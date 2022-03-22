# 创建一个新的 Scene

> Scene: 意为场景，类似于各种游戏中地点的概念，最为贴切的描述是三国志10的场景的概念。

本文将提供一个例子来告诉mod开发者如何新建一个 Scene。

```tree
data
├─position
│  └─sandbox_mainworld
│     ├─scenes
│     └─sandbox_mainworld.lua
└─...
```

一个 mod 中的场景被存放在 `data/position` 中，position目录下面的每个文件夹表示了一个 `sandbox`，sandbox 的概念可以类比于 minecraft 的主世界，地域等世界，次元的概念，设计目的是为了后续做演化模块及性能方面的考虑，现阶段只有 mainworld 这一个 sandbox。

sandbox 下有 `scenes` 文件夹，其中的每一个文件都描述了一个世界初始化时创建的 Scene 内容，以初始场景 nona_village 为例

```lua
--
-- scene_nona_village.lua
--
-- 诺娜村
--

local scene = {
    id = "nona_village",
    name = "scenes.scene_nona_village",
    backgroundUrl = "worldrpg://mods/hov_core/assets/images/scenes/nona_village.jpg",
    previewUrl = "worldrpg://mods/hov_core/assets/images/scenes/default_preview.png",
    area = 2000,
    air = {
        O2 = 2000*20,
        N2 = 2000*79,
        CO2 = 2000*1
    },
    earth = {
        soil = 90,
        stone = 10
    }
}


return scene
```

一个Scene lua文件定义了一个场景所具有的属性(property)，其属性可以被其他 systems 读取及利用来世界中的其他对象进行交互。
> 比如某些 action 可以限定只能在具有某些property的 scene 中才可以执行，比如挖掘需要在有矿藏属性的 scene 中才能执行。

其中scene的一些属性是框架定义的。
* `id`: Scene 的唯一标识，不可重复。
* `name`: Scene 的名字，会被用于显示在UI上，使用本地化文本的key，`scenes.scene_nona_village` 是在 `locale/<lang-code>/scenes.csv` 中定义的
* `backgroundUrl(optional)`: Scene 的背景图片，被用来展示在Client的主要界面上作为背景。推荐大小：1920*1080。
* `previewUrl(optional)`: Scene 的预览图片，被用来展示在Client的道路上作为Scene道路卡片的背景。推荐大小：?*?。