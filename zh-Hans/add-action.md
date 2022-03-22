# 创建一个新的 Action

> Action: 意为行为，是以 Unit 为执行者，可以对其他 Unit、Item、Scene 产生影响的一次动作，比如就寝，挖掘，钓鱼，饮用。

本文将提供一个例子来告诉mod开发者如何新建一个 Action。

```tree
data
└─actions
```

一个 mod 中的所有可被 unit 执行的 action 都被存放在 `data/actions` 中，目录下面的每个文件表示一个行为，其定义了行为的时间、限制、效果。

以挖掘(action_mine.lua)为例，

```lua
--
-- action_mine.lua
--
-- 行为：挖掘
-- 对环境
--


local action = { _version = "0.1.0" }

action.id = "mine"

action.name = "actions.mine"

action.desc = "actions.mine_desc"

action.enable = {
    actor = {
        where = {
            {
                left = {
                    type = "property",
                    value = "STR",
                },
                operation = ">",
                right = 5,
            },
            {
                left = {
                    type = "property",
                    value = "CON",
                },
                operation = ">",
                right = 5,
            }
        },
    },
    scene = {
        where = {
            {
                left = {
                    type = "property",
                    value = "miningArea",
                },
                operation = "exists"
            },
        }
    },
}

action.getDuration = function(actorUnit, args)
    return 600 - actorUnit.str - Utils.dice.roll("4d6")
end

action.progressInterval = 50

action.onStart = function(actorUnit)
    scene = Unit.getUnitPosition(actorUnit.id)
    System.console.log(Locale.T(actorUnit.name) .. "开始在" .. Locale.T(scene.name) .. "挖掘")
end

action.onProgress = function(actorUnit, tickOffset)
    System.console.log(Locale.T(actorUnit.name) .. "挖掘中...")
    local newItem = Item.createFromFactory("copper_ore")
    Item.moveItemToUnit(newItem.id, actorUnit.id)
    System.console.log(Locale.T(actorUnit.name) .. "获得了" .. Locale.T(newItem.name))
    -- TODO: add item to unit
end

action.onEnd = function(actorUnit)
    System.console.log(Locale.T(actorUnit.name) .. "挖掘结束")
    Unit.updateUnit(actorUnit.id, function(unit)
        unit.counter.mine = (unit.counter.mine or 0) + 1
        return unit
    end)
end

return action

```

一个Action lua文件定义了一个行为。

* `action.id`: Action 的唯一标识，不可重复。
* `action.name`: Action 的名字，会被用于显示在UI上，使用本地化文本的key，`actions.mine` 是在 `locale/<lang-code>/actions.csv` 中定义的
* `action.desc`: Action 的描述，会被用于显示在UI上，使用本地化文本的key，`actions.mine_desc` 是在 `locale/<lang-code>/actions.csv` 中定义的
* `action.enable`: Action 的限制条件，被用于限制一个 unit 能否执行某个 action，以action_mine为例，这个action仅在行为的执行者(actor)的 STR 以及 CON 大于 5 时才能执行，同时要求执行的地点(scene)具有 mineArea property。更多细节可查看下文。具体 enable 中筛选的语法可查看 `WorldRPG filter expression` (TODO: add doc)。
* `action.getDuration`: Action 需要的时长，WorldRPG 中1单位时间的概念为1秒。时长可根据 actor 的不同具有不同的时长。
* `action.progressInterval`: Action 多久触发一次 `action.onProgress`。
* `action.onStart`: Action 开始执行时的 hook，开发者可使用 LuaBridge API 进行游戏数据的更新及UI的更新。
* `action.onProgress`: Action 每次经过 progressInterval 后的 hook，开发者可使用 LuaBridge API 进行游戏数据的更新及UI的更新。
* `action.onEnd`: Action 结束后的 hook，开发者可使用 LuaBridge API 进行游戏数据的更新及UI的更新。一般用于结算。


## action.enable
enable 中提供的约束有 `scene`, `actor`, `toUnit`, `toItem`

> 在 WorldRPG 中 action 可以针对unit（比如交谈），也可以针对item（比如喝），也可以没有特殊的指定（比如祈祷）。这一限制主要是在 `action.enable` 中通过`toUnit`, `toItem`实现的。

### scene
针对 action 执行地点的限制，需要在 enable 中添加 `scene`
```lua
action.enable = {
    scene = {
        where = {
            {
                left = {
                    type = "property",
                    value = "miningArea",
                },
                operation = "exists"
            },
        }
    },
} 
```

### actor
针对 action 执行者的限制，需要在 enable 中添加 `actor`
```lua
action.enable = {
actor = {
        where = {
            {
                left = {
                    type = "property",
                    value = "STR",
                },
                operation = ">",
                right = 5,
            },
            {
                left = {
                    type = "property",
                    value = "CON",
                },
                operation = ">",
                right = 5,
            }
        },
    },
} 
```

### toUnit
针对unit的action，需要再 enable 中添加 `toUnit`
```lua
action.enable = {
    toUnit = true,
} 
```
如果需要额外的约束（比如property约束），可以传入 WorldRPG filter expression
```lua
action.enable = {
    toUnit = {
        where = {
            {
                left = {
                    type = "property",
                    value = "STR",
                },
                operation = "exists"
            },
        }
    },
} 
```

### toItem
针对item的action，需要再 enable 中添加 `toItem`
```lua
action.enable = {
    toItem = true,
} 
```
如果需要额外的约束（比如property约束），可以传入 WorldRPG filter expression
```lua
action.enable = {
    toItem = {
        where = {
            {
                left = {
                    type = "property",
                    value = "test",
                },
                operation = "exists"
            },
        }
    },
} 
```