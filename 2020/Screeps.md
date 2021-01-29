圣诞的时候入了Screeps，这是个MMO 编程的游戏，互相写程序强资源，但是我不太熟悉JS

在边玩边学，玩游戏的过程可以学到一些nodejs的编程知识



<!--more-->

## Basic

首先是基本内容

游戏的名称Screeps实际上是`Script the Creeps`的缩写，而`Creeps`这个词大概是小兵的意思；

游戏每一局开始，会控制一个`spawn`，这个词查了意思是卵、产卵

相当于红警之类的基地车吧（实际上这个游戏好像是模拟的星际争霸，不过没玩过不太了解）

通过spawn可以生产出creep

```js
Game.spawns['Spawn1'].spawnCreep([WORK,CARRY,MOVE],'Harvester1')
```

这个是教程里给出的生产Creep的指令

具体来看的话，`Game.spawns['Spawn1']`这里应该是指定到了具体的`spawns`，名称为`Spawn1`的这个`spawn`，之后调用了spawnCreep这个方法来生产Creep

这个函数的两个参数，第一个参数是一个列表，里面的每一项是一个模块

第二个参数是要生产的这个Creep的名称



这个列表还是有点意义的，游戏中每个Creep是有生命值的

在Creep受到攻击时会按照顺序依次失去列表中各个模块的能力

比如这个Creep如果受到一点攻击，就无法`WORK`，但是还可以 `CARRY`和`MOVE`

受到了两点攻击就只能`MOVE`了，因此对于不同用处的Creep，最好给到不同的模块、顺序



地图中存在黄色的点，是金矿一样的资源点，可以被具有`WORK`模块的Creep开采，被具有`CARRY`模块的Creep搬运



对Creep下达一个循环指令

```js
module.exports.loop = function () {
    var creep = Game.creeps['Harvester1'];
    var sources = creep.room.find(FIND_SOURCES);
    if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
        creep.moveTo(sources[0]);
    }
}
```

这样这个creep就会去到资源所在的位置开始采集，但是并不会回到Spawn，因为没有给它指定回来的函数

```js
module.exports.loop = function () {
    var creep = Game.creeps['Harvester1'];

    if(creep.store.getFreeCapacity() > 0) {
        var sources = creep.room.find(FIND_SOURCES);
        if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
            creep.moveTo(sources[0]);
        }
    }
    else {
        if( creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE ) {
            creep.moveTo(Game.spawns['Spawn1']);
        }
    }
}
```

修改的更丰富一些，对它可以存放的容量进行判断、之后回到卵的位置



这样这个creep就会一直不停的工作直到死亡，每个creep具有寿命，可以存活1500个游戏时间ticks

创建creep需要消耗不少的能量，创建出更多的creep以后

这个函数也需要进一步改进，加入一个循环令所有的creep都去工作

```js
module.exports.loop = function () {
    for(var name in Game.creeps) {
        var creep = Game.creeps[name];

        if(creep.store.getFreeCapacity() > 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0]);
            }
        }
        else {
            if(creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
                creep.moveTo(Game.spawns['Spawn1']);
            }
        }
    }
}
```

随着任务更加复杂，单独一个文件没办法组织起来工作，这时就可以创建一个新的module



新建了一个module，文件命名为`role.harvester`

```js
/*
 * Module code goes here. Use 'module.exports' to export things:
 * module.exports.thing = 'a thing';
 *
 * You can import it from another modules like this:
 * var mod = require('role.harvester');
 * mod.thing == 'a thing'; // true
 */
var roleHarvester = {

    /** @param {Creep} creep **/
    run: function(creep) {
	    if(creep.store.getFreeCapacity() > 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0]);
            }
        }
        else {
            if(creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
                creep.moveTo(Game.spawns['Spawn1']);
            }
        }
	}
};

module.exports = roleHarvester;
```

之后可以重写main模块

```js
var roleHarvester = require('role.harvester');

module.exports.loop = function () {

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        roleHarvester.run(creep);
    }
}
```

这样代码看起来就简洁很多了


