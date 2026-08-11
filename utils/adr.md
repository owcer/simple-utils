# atr v1.1

atr = add to room

Add entities to rooms, so only entities within the same room can see each other.  
Useful for simulating multiple rooms in one room, or multiple 1v1 arenas when only 1 exists, for example

`atr(entityId,roomName)`: add an entity to room so it can only see and be seen by whoever else is in the room  
`atr.seeAll(entityId,boolean)`: whether an entity should be able to see all entities (for spectators etc)

```js
globalThis.atr=(id,room)=>{
    room+=""
    if(room){
        api.setTargetedPlayerSettingForEveryone(id,"canSee",false,true)
        try{api.setEveryoneSettingForPlayer(id,"canSee",false,true)}catch(e){console.log(e)}
        atr.r[atr.i[id]] && delete atr.r[atr.i[id]][id]
        let roomObj = atr.r[room] ||= Object.create(null)
        roomObj[id]=1
        atr.i[id]=room
        let players = Object.keys(roomObj)
        for(let i = 0; i < players.length; i++){
            try{api.setOtherEntitySetting(players[i],id,"canSee",true)}catch{}
            try{api.setOtherEntitySetting(id,players[i],"canSee",true)}catch{delete roomObj[players[i]];delete atr.i[players[i]]}
        }
    } else {
        api.setTargetedPlayerSettingForEveryone(id,"canSee",true,true)
        try{api.setEveryoneSettingForPlayer(id,"canSee",true,true)}catch{}
        atr.r[atr.i[id]] && delete atr.r[atr.i[id]][id]
        let players = Object.keys(atr.i)
        for(let i = 0; i < players.length; i++){
            try{api.setOtherEntitySetting(players[i],id,"canSee",false)}catch{}
            try{api.setOtherEntitySetting(id,players[i],"canSee",false)}catch{delete atr.i[players[i]]}
        }
        try{api.setOtherEntitySetting(id,id,"canSee",true)}catch{}
    }
    let mustSee = Object.keys(atr.s)
    for(let i = 0; i < mustSee.length; i++){
        try{api.setOtherEntitySetting(mustSee[i],id,"canSee",true)}catch{delete atr.s[mustSee[i]]}
    }
}
atr.seeAll=(pid,yes)=>{
    if(api.getEntityType(pid)!=="Player")return
    if(yes){
        atr.s[pid]=1
        api.setEveryoneSettingForPlayer(pid,"canSee",true,true)
    } else {
        delete atr.s[pid]
        atr(pid,atr.i[pid])
    }
}
{
    //this block is needed because unforunately api.setEveryoneSettingForPlayer doesn't actually work for newjoiners
    let acb
    {(acb=(t,e)=>{let n=globalThis[t],o=acb.c[t]||="function"==typeof n?[n]:[],f=o.length;"function"==typeof e&&(o[f++]=e),globalThis[t]=(...t)=>{let e;for(let n=0;n<f&&(void 0===(e=o[n](...t))||!0===e);n++);return e}}).c={};}
    acb('onPlayerJoin',i=>atr(i,""))
}
atr.i=Object.create(null)//{id:roomName}
atr.r=Object.create(null)//{roomName:roomObj}
atr.s=Object.create(null)//see all list
```
