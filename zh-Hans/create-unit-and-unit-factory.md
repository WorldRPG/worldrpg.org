# 创建一个新的 Unit 及 Unit Factory

> Unit: 意为单位，类似于各个游戏中人物及怪物的可移动的概念，拥有自己的属性（property）以及可以执行某些 Action。

本文将提供一个例子来告诉mod开发者如何新建一个唯一的 Unit 或编写一个 Unit Factory 用来创建模板化的 Unit。

```tree
data
└─units
    ├─factory
    └─unique
```

一个 mod 中的唯一 unit 以及 unit factory 都被存放在 `data/units` 中，`data/units/unique`目录下面的每个文件表示一个具有唯一性的 unit，`data/units/factory`目录下的每个文件表示一个 unit factory。

## Unique Unit

以 alice 为例

```lua
--
-- unit_alice.lua
--
-- alice 人物
--

local unit = {
    id = "alice",
    name = "units.unit_alice",
    traits = {"race:human", "female"},
    portraitUrl = "worldrpg://mods/hov_core/assets/images/units/anka.jpg",
    unit = {
        STR = 5,
        CON = 5,
        DEX = 5,
        INT = 5,
        POW = 5,
        APP = 5,
        SIZ = 12,
    },
}


return unit
```

一个Unique Unit lua文件定义了一个单位所具有的属性(property)，其属性可以被其他 systems 读取及利用来世界中的其他对象进行交互。
> 比如人物的六维会影响很多 action 的效果。

其中unit的一些property是框架定义的。
* `id`: Unit 的唯一标识，不可重复。
* `name`: Unit 的名字，会被用于显示在UI上，使用本地化文本的key，`units.unit_alice` 是在 `locale/<lang-code>/units.csv` 中定义的
* `traits`: Unit 的特质，特质是 WorldRPG 框架级提供的针对 Unit 的一个标志，并提供方便的 API 供 system 查询及判断，并提供了UI显示的支持。
* `portraitUrl(optional)`: Unit 的头像图片，被用来展示在界面的上作为Unit交互卡片的背景。推荐比例：1:1。


## Unit Factory

以 jelly_chicken 为例

```lua
--
-- unit_factory_jelly_chicken.lua
--
-- 果冻鸡-unit工厂 https://steamcommunity.com/sharedfiles/filedetails/?id=1381023116
--

local unitFactory = {
    create = function()
        local id = Utils.uuid()
        return {
            id = "jelly_chicken_" .. id,
            name = "unit_jelly_chicken",
            traits = {"race:elementalCreature", "female"},
            portraitUrl = "worldrpg://mods/hov_core/assets/images/units/anka.jpg",
            unit = {
                STR = 2,
                CON = 2,
                DEX = 2,
                INT = 2,
                POW = 2,
                APP = 2,
                SIZ = 1,
            },
        }
    end
}


return unitFactory
```

一个Unit Factory lua文件定义了如何创建某种类型的 unit，factory 可被其他可以调用到 LuaBridge API 的地方（比如 systems，action）调用并创建新的 unit。

一个 factory 具有 `create()` 方法，方法返回一个 unit data。

> 详情见上 Unique Unit