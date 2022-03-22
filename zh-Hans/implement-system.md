# 实现一个 System

> System: System 是基于某些目的性对 World 中的数据进行周期性处理或扩展游戏数据的一个模块。开发者可以通过添加一个新的 System 来扩展游戏的玩法，比如天气系统。

本文将提供一个例子来告诉mod开发者如何新建一个 System。

```tree
.
└─systems
```

一个 mod 中的所有的 system 都被存放在 `systems` 文件夹中，目录下面的每个文件表示一个系统。并需要在 `module.lua` 中进行添加。

```lua
--
-- module.lua
--

local module = { _version = "0.1.0" }
module.systems = {
    "fantasy_unit_system",
    "sleep_system",
    "fantasy_position_system",
    "equipment_system",
    "topoclimate_system",
    "weather_system",
}

-- ...
-- ...

return module
```

一个system包括

* `system.worldInit()`: 在世界创建时会执行，可调用 LuaBridge API。
* `system.sceneCreateInterceptor`: Scene 的创建拦截器，可以在 Scene 初始化或后续动态创建的过程中添加没有在数据文件中声明的 property。
* `system.unitCreateInterceptor`: Unit 的创建拦截器，可以在 Unit 初始化或后续动态创建的过程中添加没有在数据文件中声明的 property。
* `system.itemCreateInterceptor`: Item 的创建拦截器，可以在 Item 初始化或后续动态创建的过程中添加没有在数据文件中声明的 property。
* `system.sandboxCreateInterceptor（暂未实现）`: Sandbox 的创建拦截器，可以在 Sandbox 初始化或后续动态创建的过程中添加没有在数据文件中声明的 property。
* `system.onSceneCreated（暂未实现）`: TODO
* `system.onSandboxCreated（暂未实现）`: TODO
* `system.onUnitCreated（暂未实现）`: This function will called when `unitCreated` event get trigger, this event will be triggered when unit get created. unlike the `unitCreateInterceptor`, the return type is nil and will not affect the world.
* `system.onUnitDead（暂未实现）`: This function will called when `unitDead` event get trigger, this event will be triggered when unit dead
* `system.onActionStart（暂未实现）`: TODO
* `system.onActionEnd（暂未实现）`: TODO
* `system.onTick_xxx`: 基于时间的钩子，会在时间经过xxx之后调用。
    * `system.onTick`: called every tick, 1 second if 1 tick = 1 second, **Please don't put heavy calculation task in this callback**.
    * `system.onTick_10`: called every 10 ticks, 10s if 1 tick = 1 second, **Please don't put heavy calculation task in this callback**.
    * `system.onTick_30`: called every 30 ticks, 30s, if 1 tick = 1 second, **Please don't put heavy calculation task in this callback**.
    * `system.onTick_60`: called every 60 ticks, 1 minute, if 1 tick = 1 second, **Please don't put heavy calculation task in this callback**.
    * `system.onTick_300`: called every 300 ticks, 5 minute, if 1 tick = 1 second
    * `system.onTick_3600`: called every 3600 ticks, 1 hour, if 1 tick = 1 second
    * `system.onTick_86400`: called every 86400 ticks, 1 day, if 1 tick = 1 second
    * `system.onTick_604800`: called every 604800 ticks, 1 week, if 1 tick = 1 second

下面以 weather_system 为例分别说明一个 system 所包含的内容以及 WorldRPG 是如何利用 system 这种模块进行游戏内容的扩展的

## Weather System
```lua
--
-- weather_system.lua
--
-- weather system, 提供了通用的天气系统。
-- 提供 component: scene.weather
--

local system = { _version = "0.1.0" }

system._getWeatherByTag = function(tag)
    error("not implemented")
end

system.worldInit = function ()
    -- run when world init
end

system.sceneCreateInterceptor = function(scene)
    local weathers = Data.loadJson("extra/weather.json")
    if scene.weather == nil then
        scene.weather = weathers[1].key -- fuck lua, array start from 1...
    end
    return scene
end

system.onUnitCreated = function(unit)
    return nil
end

system.onUnitDead = function(unit)
    return nil
end

system.onTick_3600 = function(tick)
    System.console.debug("[weather_system] updating onTick_3600")
    local scenes = Scene.getAllScenes()
    for sceneId, scene in pairs(scenes) do
        local weatherDiceResult = Utils.dice.roll("1d100")
        local nextWeather = scene.weather
        if weatherDiceResult > 99 then
            nextWeather = "Stormy"
        end
        -- TODO: add transition between weather
    end
    System.console.debug("[weather_system] updated onTick_3600")
end

return system

```

Weather System 会为游戏中的 Scene 添加一个新的 property `weather`，不同的 weather 可以为游戏的 action 或其他系统产生影响，比如种植系统。

```lua
system.sceneCreateInterceptor = function(scene)
    local weathers = Data.loadJson("extra/weather.json")
    if scene.weather == nil then
        scene.weather = weathers[1].key -- fuck lua, array start from 1...
    end
    return scene
end
```

weather system 中的 `sceneCreateInterceptor` 会在 scene 初始化或者动态创建的时候拦截并注入一些 property，上面的代码在场景创建的过程中注入了 `weather` 属性。其初始值是通过加载mod中`data/extra/weather.json` 并获取第一个值来实现的。

```json
// weather.json
[{
    "key": "Cloudy",
    "tags": ["common"]
}, {
    "key": "Sunny",
    "tags": ["common", "hot"]
}, {
    "key": "Rain",
    "tags": ["rainy", "cool"]
}, {
    "key": "Drizzle",
    "tags": ["rainy", "cool"]
}, {
    "key": "Snow",
    "tags": ["challenge", "cold"]
}, {
    "key": "Stormy",
    "tags": ["challenge", "cold"]
}]
```

同时 weather system 会周期性的变更所有场景中的 weather property 来实现天气的变化。

```lua
system.onTick_3600 = function(tick)
    System.console.debug("[weather_system] updating onTick_3600")
    local scenes = Scene.getAllScenes()
    for sceneId, scene in pairs(scenes) do
        local weatherDiceResult = Utils.dice.roll("1d100")
        local nextWeather = scene.weather
        if weatherDiceResult > 99 then
            nextWeather = "Stormy"
        end
        -- TODO: add transition between weather
    end
    System.console.debug("[weather_system] updated onTick_3600")
end
```

onTick_3600 相当于每1小时游戏时间会执行一次，Weather System 中遍历所有的 Scene 后并按照一些随机的方式更新 Scene 中的 weather 属性。