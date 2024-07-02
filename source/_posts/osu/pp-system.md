---
title: 深入浅出剖析osu!的PP算法
date: 2024-07-01 9:29:59
categories: [osu]
tags:
    - osu!
    - PP算法
---

本文是笔者研究osu!戳泡泡模式PP算法的一个记录文章, 旨在帮助其他对PP算法感兴趣的玩家快速建立对PP算法的认知. 由于大部分内容是笔者的个人理解和经验谈, 可能存在错误或者偏差, 欢迎读者提出改进意见和补充内容.

本文的代码片段来源于rosu-pp, rosu-pp是osu!lazer全模式算法的Rust实现, 也是目前最流行的PP计算库. 本文面向略有编程经验的读者, 读者不必深入了解Rust, 只需要具有本科级别的编程知识和数学知识即可理解本文内容.

# 初探PP算法

## 两步式计算

抛开复杂的算法不谈, 我们先来谈论一些表象. 在表面来看, PP系统会综合评估**谱面难不难**和**打的好不好**两个概念, 最终计算出一个PP数值.

广泛存在的一个误区是PP系统会根据Replay来评判这两个概念, 这是一种错误的认知. 实际上, PP计算是**两步式**进行的, 第一步计算谱面难度(这一步通常自变量是所有物件和速度倍率), 第二步是根据第一步算出的**谱面难度**和以及一些**其他变量**(例如谱面的三维, 玩家Acc, Miss数, 最大Combo)共同计算出最终PP数值.

不难想象, 第一步计算的时候我们主要在考虑物件之间相互影响产生的基值, 而第二步计算主要是根据玩家的表现在基值上进行倍率的调整. 这里我们举一个简单的例子, 曲奇在FDFD中取得了939PP的成绩, 而mrekk取得了926PP的成绩, Accolibed取得了640PP的成绩, 他们三人在第一步计算中算出的基值是**相同**的, 只是在第二步计算中产生了不同的倍率. 曲奇的准确度高于mrekk, 固取得了更高的acc_bonus. Accolibed掉了一个Miss, 固得到了miss_penalize. 最终形形色色的bonus和penalize在第二步作用在基值上, 算出了最终的数值.

也正因如此, 在利用各种PP计算器来计算大量成绩的时候, 先按照谱面归类将大幅提高计算效率, 因为在没有开启调整速度的Mod的情况下, 第一步计算(这也是决速步骤)只用进行一次就足够了.

> 上文提到的误区实际上源自于玩家对算法的过度期待: Miss了简单的部分, SS了困难的部分, PP算法是无法分辨出这两个场景的. 这些场景只能被视为一个整体, 表现于上文提到的**其他变量**中, 影响第二步计算的各种倍率. 这也是各个PP计算器不需要提供Replay, 只需要提供最终结果, 就能计算出准确PP的原理所在.

至此, 我们已经将PP计算的过程分为了两部分, 通常来说, 第一步被称为Difficulty计算, 第二步被称为Performance计算.

![Snipaste_2024-07-01_11-34-12.png](https://s2.loli.net/2024/07/01/JE8c1DUvZdnLy7g.png)

## Difficulty 与 Skills

经过上文的探究, 我们明白了Difficulty的计算是重中之重, 是区分**谱面难不难**的关键. 

osu!的设计者们有意识地从多个维度来评估谱面的难度, 在多年的逐步迭代中明确了Skills这个概念. 当然, 针对不同的模式, Skill分别有不同维度的定义.

对于戳泡泡玩家而言, 戳泡泡模式的Skills应该至少包含了Aim和Speed这两个老生常谈的维度, 当然事实也是这样的, 可以参考下面的表格.

- osu!standard: Aim, Speed, Flashlight
- osu!mania: Strain
- osu!catch: Movement
- osu!taiko: Color, Rhythm, Stamina

这里由于笔者对其他模式不甚了解, 这里我们主要将目光聚焦于osu!standard. 本文主要也将谈论Aim, Speed两个Skill.

## Skill 与 微积分?

相信读者读到这里已经对Aim, Speed的具体计算非常感兴趣了, 但在正式开始之前, 我们先笼统地认识一下Skill的计算方式.

读者不妨设想一下, 对于一张时长几分钟的谱面, 我们应该如何计算他的难度呢? 实际上这个问题是困难的, 物件以时间为轴堆叠仅仅是一堆表示位置的数组, 是很难衡量困难与否的. 如果在本科期间修读过高等数学的读者可能对这个例子感到熟悉: 一个光滑的曲面本身可能是不规则的, 想要计算面积是很困难的, 但是如果将曲面无限细分, 将最小单位看作是一个正方体, 问题就很好解决了. 即整体的求解是困难的, 但是一个切片往往是容易求解的.

Skill的计算遵循了类似先*微分*后*积分*的过程, 在计算过程中, 物件被**细分**进行**单独对待**.

简单来说, 每个物件都可以算作一个表示难度的元素, 在PP系统中, 这个元素被称作是**Strain**. 某Skill数值的计算依托于一个Strain构成的**集合**. 最终Skill数值(例如: raw_aim, raw_speed), 实际上是对Strains进行积分(以某种方式累加)的结果.

戳泡泡中, 大部分Pattern(例如: 锐角跳, 钝角跳)都是针对**Strain**展开的, 即我们在计算单个**Strain**的过程中区分这些Pattern, 并给出最终的Strain值. 在实际计算Strain的过程中, 我们可以拿到很多参数辅助我们, 决定"当前物件的难度".

- osu_curr_obj, osu_last_obj, osu_last_last_obj: 拿到前后的物件
- travel_dist, travel_time, strain_time, jump_dist: 距离, 时间等几何量、物理量

我们会在之后的讲解中逐步接触和理解这些概念. 在之后的讲解中, 我们也将以Strain计算为重点, **自下而上**地剖析具体计算过程.

接下来, 我们将逐步探寻Aim, Speed两个Skill, 并且尝试阅读单个Strain是如何被计算的.

# 探索Aim Skill

在rosu-pp中, 我们可以在/osu/difficulty/skills目录找到所有Skill的定义和实现. 这里我们尝试开始阅读aim.rs的源代码

```rust
impl<'a> Skill<'a, Aim> {
    fn strain_value_at(&mut self, curr: &'a OsuDifficultyObject<'a>) -> f64 {
        self.inner.curr_strain *= strain_decay(curr.delta_time, STRAIN_DECAY_BASE);
        self.inner.curr_strain +=
            AimEvaluator::evaluate_diff_of(curr, self.diff_objects, self.inner.with_sliders)
                * SKILL_MULTIPLIER;

        self.inner.curr_strain
    }
}
```

在针对Skill<'a, Aim>的实现中, 最吸引我们注意的是**strain_value_at**方法, 因为**curr_strain**值正是从这里发源的.

在上文的解释中, 我们已经明确了一个概念: 在计算过程中, 物件被**细分**进行**单独对待**. 

所以显然, strain_value_at会被调用多次, 其参数curr表示将要被计算的某一物件, self.diff_objects是所有物件列表的一个引用.

> self.diff_objects的存在是必要的, 因为在计算单个物件的时候, 我们需要获取该物件的**上下文**. 
> ```rust
> pub trait IDifficultyObject: Sized {
>    fn idx(&self) -> usize;
>    fn previous<'a, D>(&self, backwards_idx: usize, diff_objects: &'a [D]) -> Option<&'a D> {}
>    fn next<'a, D>(&self, forwards_idx: usize, diff_objects: &'a [D]) -> Option<&'a D> {}
>}
> ```
> 简单阅读IDifficultyObject的定义, 相信读者已经明白idx, diff_objects的必要性和使用场景了.

接下来, 我们将目光放到**AimEvaluator::evaluate_diff_of**方法上, 我们将按行分块解释这个方法.

## osu_curr_obj

```rust
let osu_curr_obj = curr;

let Some((osu_last_last_obj, osu_last_obj)) = curr
    .previous(1, diff_objects)
    .zip(curr.previous(0, diff_objects))
    .filter(|(_, last)| !(curr.base.is_spinner() || last.base.is_spinner()))
else {
    return 0.0;
};
```

这部分主要负责解构出osu_curr_obj, osu_last_obj, osu_last_last_obj. 

利用diff_objects和对象储存的物件ID就可以还原出上一个物件与下一个物件.

这里很明显地, 当物件为转盘(Spinner)时, 方法将直接返回0.0, 不执行后续计算.

## 距离/时间=速度

```rust
// * Calculate the velocity to the current hitobject, which starts
// * with a base distance / time assuming the last object is a hitcircle.
let mut curr_vel = osu_curr_obj.lazy_jump_dist / osu_curr_obj.strain_time;

// * But if the last object is a slider, then we extend the travel
// * velocity through the slider into the current object.
if osu_last_obj.base.is_slider() && with_sliders {
    // * calculate the slider velocity from slider head to slider end.
    let travel_vel = osu_last_obj.travel_dist / osu_last_obj.travel_time;
    // * calculate the movement velocity from slider end to current object
    let movement_vel = osu_curr_obj.min_jump_dist / osu_curr_obj.min_jump_time;

    // * take the larger total combined velocity.
    curr_vel = curr_vel.max(movement_vel + travel_vel);
}

// * As above, do the same for the previous hitobject.
let mut prev_vel = osu_last_obj.lazy_jump_dist / osu_last_obj.strain_time;

if osu_last_last_obj.base.is_slider() && with_sliders {
    let travel_vel = osu_last_last_obj.travel_dist / osu_last_last_obj.travel_time;
    let movement_vel = osu_last_obj.min_jump_dist / osu_last_obj.min_jump_time;

    prev_vel = prev_vel.max(movement_vel + travel_vel);
}

let mut wide_angle_bonus = 0.0;
let mut acute_angle_bonus = 0.0;
let mut slider_bonus = 0.0;
let mut vel_change_bonus = 0.0;

// * Start strain with regular velocity.
let mut aim_strain = curr_vel;
```

这部分的主要逻辑是利用距离除以时间算出速度, 并且把速度当做初始aim_strain值.

这里我们解释一些**基本概念**, 以便于具体理解这些参数:

---

### strain_time

表示当前物件与上一物件的时间间隔, 单位为毫秒.

计算方式为: `当前物件的起始时间 - 上一物件的起始时间`.

也可以用下面的这种方式进行计算: `(60 / BPM) * 节拍细分 * 1000`.

对于大部分谱面, 符合下面这样的例子: 

- 对于180BPM的跳: (60 / 180) * 1/2 * 1000 = 166.67ms.
- 对于180BPM的串: (60 / 180) * 1/4 * 1000 = 83.34ms.
- 对于200BPM的跳: (60 / 200) * 1/2 * 1000 = 150.00ms.
- 对于200BPM的串: (60 / 200) * 1/4 * 1000 = 75.00ms.

> 算法中规定, MIN_DELTA_TIME = 25.0, 即strain_time的最小值为25ms.
>
> 关于节拍细分的更多内容可以查看osu!Wiki: [音符时值 (Beat Snap Divisor)](https://osu.ppy.sh/wiki/zh/Client/Beatmap_editor/Beat_snap_divisor)

### lazy_jump_dist

表示当前物件与上一物件的标准化距离, 单位为osu!pixel

计算方式为: normalize(`||当前物件的位置 - 上一物件的位置||`)

> [osu!pixel](https://osu.ppy.sh/wiki/zh/Client/Playfield)的区域为(0, 0)到(512, 384), 在实际游玩中会根据分辨率进行缩放.
>
> 物件的位置会进行标准化, 标准化的时候, 取圆圈物件的半径为50单位长度, 注: 此时会考虑CS.
>
> 同时也与作图的[堆叠度](https://osu.ppy.sh/wiki/zh/Beatmap/Stack_leniency)配置有关, 感兴趣的读者可以自行阅读相关代码

### travel_dist

表示**滑条物件**的滑行距离

### travel_time

表示**滑条物件**的滑行时间

### min_jump_time

表示刨除滑行时间后, 当前物件与上一滑条**头**之间的时间间隔.

**通常在上一物件为滑条的情形下使用**, 如果上一物件不是滑条, 该值与strain_time**相等**.

### min_jump_dist

表示刨除滑行距离后, 当前物件与上一滑条**尾**之间的距离.

**通常在上一物件为滑条的情形下使用**, 如果上一物件不是滑条, 该值与lazy_jump_dist**相等**.

---

接下来我们返回到代码解析部分, 相信读者最感兴趣的就是extend velocity这部分, 我们画图来解释一下

![](https://s2.loli.net/2024/07/02/WBSebI6OzCKnd3a.png){width="720px"}

请读者将**圆圈2**当做当前物件, 很明显, 绿色的是第一步计算的curr_vel, 接下来, 进入if逻辑的判断.

因为上一个物件是滑条, 条件满足, 这里接下来会分别计算红色部分和橘色部分的速度, 并且求和, 与原curr_vel取最大值.

很明显, 对于图中这种排列, 红色部分和橘色部分的速度和是大于绿色的, 故采用后续计算出的值作为curr_vel.

这里客观来讲, extend velocity更能表现出**滑条1与圆圈2组合**的速度.

## 锐角与钝角排列

在上一部分, 我们根据速度计算出了Strain的基础值, 接下来PP算法考虑了经典的锐角和钝角Pattern, 计算出了有针对性的奖励系数.

注: 此处的代码经过了一些删减、重新排序和提取, 在不改变逻辑的情况下提升了可读性.

这里我们按照注释, 将代码分为四部分: 节奏判断、基础值计算、锐角增益计算、重复惩罚.

```rust
// * If rhythms are the same (节奏判断).
if osu_curr_obj.strain_time.max(osu_last_obj.strain_time)
    < 1.25 * osu_curr_obj.strain_time.min(osu_last_obj.strain_time)
{
    // * Rewarding angles, take the smaller velocity as base (角度基础值计算).
    let angle_bonus = curr_vel.min(prev_vel);

    wide_angle_bonus = Self::calc_wide_angle_bonus(curr_angle);
    acute_angle_bonus = Self::calc_acute_angle_bonus(curr_angle);

    // * Only buff deltaTime exceeding 300 bpm 1/2 (锐角增益计算).
    if osu_curr_obj.strain_time > 100.0 {
        acute_angle_bonus = 0.0;
    } else {
        let base1 =
            (FRAC_PI_2 * ((100.0 - osu_curr_obj.strain_time) / 25.0).min(1.0)).sin();

        let base2 = (FRAC_PI_2
            * ((osu_curr_obj.lazy_jump_dist).clamp(50.0, 100.0) - 50.0)
            / 50.0)
            .sin();

        // * Multiply by previous angle, we don't want to buff unless this is a wiggle type pattern.
        acute_angle_bonus *= Self::calc_acute_angle_bonus(last_angle)
        // * The maximum velocity we buff is equal to 125 / strainTime
            * angle_bonus.min(125.0 / osu_curr_obj.strain_time)
            // * scale buff from 150 bpm 1/4 to 200 bpm 1/4
            * base1.powf(2.0)
                // * Buff distance exceeding 50 (radius) up to 100 (diameter).
            * base2.powf(2.0);
    }

    // * (重复惩罚计算)

    // * Penalize wide angles if they're repeated, reducing the penalty as the lastAngle gets more acute.
    wide_angle_bonus *= angle_bonus
        * (1.0
            - wide_angle_bonus.min(Self::calc_wide_angle_bonus(last_angle).powf(3.0)));
    // * Penalize acute angles if they're repeated, reducing the penalty as the lastLastAngle gets more obtuse.
    acute_angle_bonus *= 0.5
        + 0.5
            * (1.0
                - acute_angle_bonus
                    .min(Self::calc_acute_angle_bonus(last_last_angle).powf(3.0)));
}
```

### 节奏判断

在最外层进行了节奏的判断, 只有符合条件的物件会得到额外的Bonus. 这里主要是锐角和钝角的Bonus.

首先, 算法先保证了节奏变化幅度不大, 限制当前物件与上一物件的**间隔时间变化在25%以内**.

在客观角度考虑, Pattern存在的必要条件就是节奏变化幅度较小. 如果节奏变化太大就不能称为是Pattern了.

### 角度基础值计算

```rust
wide_angle_bonus = Self::calc_wide_angle_bonus(curr_angle);
acute_angle_bonus = Self::calc_acute_angle_bonus(curr_angle);

fn calc_wide_angle_bonus(angle: f64) -> f64 {
    (3.0 / 4.0 * ((5.0 / 6.0 * PI).min(angle.max(PI / 6.0)) - PI / 6.0))
    .sin()
    .powf(2.0)
}

fn calc_acute_angle_bonus(angle: f64) -> f64 {
    1.0 - Self::calc_wide_angle_bonus(angle)
}
```

接下来, 算法根据物件之间角度的数值, 计算出了钝角和锐角增益的基础值, 范围为0 ~ 1.

`((5.0 / 6.0 * PI).min(angle.max(PI / 6.0)) - PI / 6.0)` 计算了角度与`π/6`的偏移值.

`3/4 * 偏移值` 是在对定义域进行伸缩, 伸缩到(30, 150).

`sin²(偏移值)` 将结果限制到0 ~ 1之间

使用GeoGebra可以画出下面这样的曲线, 红线代表钝角曲线, 蓝色代表锐角曲线.

![Snipaste_2024-07-02_21-42-25.png](https://s2.loli.net/2024/07/02/8i6cwDeaE2trk57.png){width="720px"}

在客观角度考虑, 30度以上, 钝角增益系数随角度平滑增长. 150度以下, 锐角增益系数随角度平滑增长, 是很自然的设计.

### 锐角增益计算

首先, 锐角增益中`osu_curr_obj.strain_time > 100.0`限制了只有300BPM以上的跳或者150BPM以上的串会被增益.

这意味着**大部分的排列不会吃到锐角增益**. 这个结论很有意思, 也很符合逻辑. 300BPM以上的跳不必多说了.

这里150BPM的串看似几乎没有限制, 但是请读者考虑一下, 小于90度的串甚至已经进入了aim control的领域了.

> 可能有读者对 acute_angle_bonus 置为 0 有疑问, 在下文的讲解中, 这个顾虑便会消失.
>
> 实际上, 最终增益系数的值是选取锐角和钝角增益的**较大者**, 也就说一个物件要么被认为是钝角、要么被认为是锐角.

```rust
let base1 =
    (FRAC_PI_2 * ((100.0 - osu_curr_obj.strain_time) / 25.0).min(1.0)).sin();

let base2 = (FRAC_PI_2
    * ((osu_curr_obj.lazy_jump_dist).clamp(50.0, 100.0) - 50.0)
    / 50.0)
    .sin();

// * Multiply by previous angle, we don't want to buff unless this is a wiggle type pattern.
acute_angle_bonus *= Self::calc_acute_angle_bonus(last_angle)
// * The maximum velocity we buff is equal to 125 / strainTime
    * angle_bonus.min(125.0 / osu_curr_obj.strain_time)
    // * scale buff from 150 bpm 1/4 to 200 bpm 1/4
    * base1.powf(2.0)
    // * Buff distance exceeding 50 (radius) up to 100 (diameter).
    * base2.powf(2.0);
```

这里英文注释为我们理解提供了极大便利.

首先`*= Self::calc_acute_angle_bonus(last_angle)`, 这个操作实际上在**规避"离群值"**.

这里注释中提到, 只希望考虑摇摆型的锐角, 摇摆型可以理解为: 连续的几个物件都符合锐角的特征.

而偶然性的, 突发性的锐角不在增益范围, 这也是合情合理的, 这种突发性的锐角不应被视作Pattern.

接着, 对velocity, strain_time, distance分别进行了伸缩.

base1的增益范围为: 150 ~ 200BPM的串 (1/4节拍)

base2的增益范围为: 互过圆心的物件 ~ 正好相切的物件 (50单位 ~ 100单位)

### 重复惩罚计算

```rust
// * Penalize wide angles if they're repeated, reducing the penalty as the lastAngle gets more acute.
wide_angle_bonus *= angle_bonus
    * (1.0
        - wide_angle_bonus.min(Self::calc_wide_angle_bonus(last_angle).powf(3.0)));
// * Penalize acute angles if they're repeated, reducing the penalty as the lastLastAngle gets more obtuse.
acute_angle_bonus *= 0.5
    + 0.5
        * (1.0
            - acute_angle_bonus
                .min(Self::calc_acute_angle_bonus(last_last_angle).powf(3.0)));
```

经过逻辑分析后, 我们可以将惩罚的关键部分抽象为`1.0 - f(x)`, f(x)的增大意味着受到更大的惩罚.

这里通过比较最后一个角度的Bonus, 可以间接地对重复的钝角进行惩罚.

如果最后一个角度更锐利, 惩罚因子f(x)会减小, 从而对重复的宽角度施加更轻微的惩罚, 间接地处理重复的角度情况.

`calc_wide_angle_bonus` 函数的返回值通过 `powf(3.0)` 进行了幂运算, 可能是为了增加惩罚因子的幅度.

`min()` 它选择较小的值作为惩罚因子, 这个比较是为了确保惩罚因子不会超过 wide_angle_bonus 的值.

## 速度变化

上一部分我们针对锐角和钝角排列进行了奖励, 其前提是速度相近, 即速度变化在25%以内.

下面我们将引入速度变化奖励, 这次与上文有着相反的前提, 即速度一定要有不同, 我们将采用与上一节类似的策略来分析.

这里我们按照注释, 将代码分为三部分: 基础值计算、重叠奖励、时间惩罚.

```rust
if prev_vel.max(curr_vel).not_eq(0.0) {
    // * (基础值计算)
    // * We want to use the average velocity over the whole object when awarding
    // * differences, not the individual jump and slider path velocities.
    prev_vel = (osu_last_obj.lazy_jump_dist + osu_last_last_obj.travel_dist)
        / osu_last_obj.strain_time;
    curr_vel =
        (osu_curr_obj.lazy_jump_dist + osu_last_obj.travel_dist) / osu_curr_obj.strain_time;

    // * Scale with ratio of difference compared to 0.5 * max dist.
    let dist_ratio_base =
        (FRAC_PI_2 * (prev_vel - curr_vel).abs() / prev_vel.max(curr_vel)).sin();
    let dist_ratio = dist_ratio_base.powf(2.0);

    // * Reward for % distance up to 125 / strainTime for overlaps where velocity is still changing. (重叠奖励)
    let overlap_vel_buff = (125.0 / osu_curr_obj.strain_time.min(osu_last_obj.strain_time))
        .min((prev_vel - curr_vel).abs());

    vel_change_bonus = overlap_vel_buff * dist_ratio;

    // * Penalize for rhythm changes. (时间惩罚)
    let bonus_base = (osu_curr_obj.strain_time).min(osu_last_obj.strain_time)
        / (osu_curr_obj.strain_time).max(osu_last_obj.strain_time);
    vel_change_bonus *= bonus_base.powf(2.0);
}
```

### 基础值计算

首先, 针对`prev_vel`和`curr_vel`进行了重算, 使用了之前图1中**绿色**的部分作为速度值.

