
[How I build games with Entitas (FNGGames) · sschmid/Entitas Wiki](https://github.com/sschmid/Entitas/wiki/How-I-build-games-with-Entitas-\(FNGGames\))

你用了不合适的框架去做不合适的事情，如果你做的是一个战斗场景内单位众多的游戏，用[ECS](https://zhida.zhihu.com/search?content_id=633151584&content_type=Answer&match_order=1&q=ECS&zhida_source=entity)写战斗逻辑是一点毛病都没有的。但是首先用一个框架之前先搞明白它的应用场景，它能为你解决什么方面的问题，它的业务边界是什么？即什么方面的逻辑是该由它负责的，什么方面的逻辑它是不适合的。

特别是以上这个图，看看ECS应该如何与外部服务配合的，使用ECS框架开发游戏战斗逻辑，并非让你用ECS的开发模式去做一切游戏逻辑，甚至还有想拿它做UI，更是离谱。从你提问中谈到的技能系统来说，首先你得整个技能组织、配置、伤害计算等等，你都可以完全基于面向对象来设计，比如有IAbility（技能）、ITarget（目标）、ITargetSelector（目标选择器）、Modifier（Buf、Debuf、光环、法球）、Action（具体行为，比如FireSoundAction、ApplyModifierAction、FireEffectAction、FireDamageAction等等）、AbilityContext，然后还得有AbilityEvent和ModifierEvent，将这些概念抽象出来，面向对象去设计技能系统，这些部分可以最终抽象为一套无状态的服务，然后才是跟ECS结合的部分，所有的状态，都必须存储在ECS的组件中（Component）。当释放技能时，ECS的System调用无状态的技能服务去读取一个具体的技能Ability，创建一个AbilityContext（最好是ECS组件），技能上下文接收游戏中的各种事件，触发技能Action或者Modifier中的的Action，执行播放人为动作，播放效果、挂buf、减伤等动作。这些Action一部分是需要跟视图相关服务配合，一部分是修改ECS系统组件值或者增删组件。Ability、Modifier都是基于面向对象设计，但是每次释放产生的实例AbilityEntity、ModifierEntity都应该是ECS的组件。


我们项目使用的就是MVVM + ECS，上面提到的也大概是我们项目技能系统的设计，希望对你有帮助。