# 快速开始

作为 WorldRPG mod 的开发者，首先需要先获取一个 Client，加群可获取最新的 Client 用于游玩和测试，群号 829818564。

Client 用于加载游戏数据并显示游戏交互画面，其会加载 Client 根目录下的 mods 文件夹进行所有数据以及逻辑的加载。

## 游戏数据与逻辑
> WorldRPG 参考了面向数据的设计方式。

WorldRPG 的框架下一个 World（世界，可以理解为一组存档数据+逻辑）包含了以下数据概念
* Scene: 意为场景，类似于各种游戏中地点的概念，最为贴切的描述是三国志10的场景的概念。
* Unit: 意为单位，类似于各个游戏中人物及怪物的可移动的概念，拥有自己的属性（property）以及可以执行某些 Action。
* Item: 意为道具，可以被 Unit 带有或放置于某个 Scene 的不可移动的概念，拥有自己的属性（property）。
* Story: 意为

同时 WorldRPG 具有以下逻辑概念
* Action: 意为行为，是以 Unit 为执行者，可以对其他 Unit、Item、Scene 产生影响的一次动作，比如就寝，挖掘，钓鱼，饮用。
* System: 参考了面向数据的设计方式，System 是基于某些目的性对 World 中的数据进行处理的概念。开发者可以通过添加一个新的 System 来扩展游戏的玩法，比如天气系统。


## Mod 结构

```tree
.
└─hov_core
    ├─assets
    │  ├─audios
    │  └─images
    │     ├─items
    │     ├─scenes
    │     └─units
    ├─data
    │  ├─actions
    │  ├─extra
    │  ├─items
    │  │  ├─factory
    │  │  └─unique
    │  ├─position
    │  │  └─sandbox_mainworld
    │  │      └─scenes
    │  ├─stories
    │  └─units
    │      ├─factory
    │      └─unique
    ├─locale
    │  ├─de
    │  ├─en
    │  ├─fr
    │  ├─ja
    │  ├─zh-Hans
    │  │  ├─extra
    │  │  └─stories
    │  └─zh-Hant
    ├─systems
    └─webview
```