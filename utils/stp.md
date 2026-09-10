# stp - simple timeouts polyfill

Usage:
- `setTimeout(cb,ms,...args)` returns `id` executes `cb(...args)` after `ms` milliseconds
- `setInterval(cb,ms,...args)` returns `id` executes `cb(...args)` every `ms` milliseconds
- `setImmediate(cb,ms,...args)` returns `id` executes `cb(...args)` the very next tick
- where `cb` can either be a `function` (recommended), or a `string` to be `eval`ed
- `clearTimeout(id)`
- `clearInterval(id)`
- `clearImmediate(id)`

Code:
Minified:
```js
//stp (v2) - simple timeouts polyfill - set and clear timeouts and intervals
//made by Ocelote - licensable under CC0 - public domain - no credit required
//setTimeout and cousins work seamlessly
globalThis.setTimeout=(e,t,...i)=>{if("string"==typeof e){let t=e;e=()=>(0,eval)(t)}let o=t/50>>>0,s=setTimeout.c+o;setTimeout.d[s]||=[];let l=setTimeout.d[s];return l[l.length]=[()=>e(...i),setTimeout.i],setTimeout.r[setTimeout.i]=[l,l.length-1],setTimeout.i++},globalThis.clearTimeout=e=>{let t=setTimeout.r[e];t&&(delete t[0][t[1]],delete setTimeout.r[e])},globalThis.setInterval=(e,t,...i)=>{if("string"==typeof e){let t=e;e=()=>(0,eval)(t)}let o=t/50>>>0;o=o>1?o:1;let s=setTimeout.c+o;setTimeout.d[s]||=[];let l=setTimeout.d[s];return l[l.length]=[()=>e(...i),setTimeout.i,o],setTimeout.r[setTimeout.i]=[l,l.length-1],setTimeout.i++},globalThis.clearInterval=e=>{clearTimeout(e)},globalThis.setImmediate=(e,...t)=>setTimeout(e,0,...t),globalThis.clearImmediate=e=>{clearTimeout(e)},globalThis.queueMicroTask=e=>{queueMicroTask},setTimeout.d={},setTimeout.r={},setTimeout.q={};{let e=.001,t=1e3/20,i=t,o=1-e,s=Date.now();setTimeout.t=()=>{let l=Date.now();for(i+=(l-s)*e,s=l;i>=t;){s=l,i*=o;let e=setTimeout.d[setTimeout.c];if(!e)return setTimeout.c++;e.forEach((t,i)=>{let o=t[0],s=t[1];try{o()}catch(e){api.broadcastMessage((t[2]?"Interval error: ":"Timeout error: ")+e.name+": "+e.message,{color:"#ff9d87"})}if(t[2]&&setTimeout.d[setTimeout.c][i]){let e=t[2],i=setTimeout.c+e;setTimeout.d[i]||=[];let l=setTimeout.d[i];l[l.length]=[o,s,e],setTimeout.r[s]=[l,l.length-1]}else delete setTimeout.r[s];delete e[i]}),delete setTimeout.d[setTimeout.c],setTimeout.c++}}}setTimeout.c=0,setTimeout.i=0,tick=()=>{setTimeout.t()};
```

Unminified:
```js
//stp (v2) - simple timeouts polyfill - set and clear timeouts and intervals
//made by Ocelote - licensable under CC0 - public domain - no credit required
//setTimeout and cousins work seamlessly

//explicitly using globalThis to make it behave correctly in strict mode :D
globalThis.setTimeout = (cb,ms,...args) => {
    if(typeof cb==="string"){
        let s = cb
        cb=()=>(0,eval)(s)
    }
    let ticks = ms/50 >>> 0 //tick runs every 50ms
    let usetick = setTimeout.c+ticks
    setTimeout.d[usetick] ||= []
    let usespot = setTimeout.d[usetick]
    usespot[usespot.length] = [()=>cb(...args),setTimeout.i] //using instead of push for interruption-proofness
    setTimeout.r[setTimeout.i] = [usespot,usespot.length-1]
    return setTimeout.i++
}
globalThis.clearTimeout = id => {
    let arr = setTimeout.r[id]
    if(!arr) return //does nothing if invalid id
    let spot = arr[0]
    let idx = arr[1]
    delete spot[idx] //make the forEach not iterate over it
    delete setTimeout.r[id]
}
globalThis.setInterval = (cb,ms,...args) => {
    if(typeof cb==="string"){
        let s = cb
        cb=()=>(0,eval)(s)
    }
    let ticks = ms/50 >>> 0
    ticks = ticks > 1 ? ticks : 1 //ensure it's at least 1 to prevent infinite loops
    let usetick = setTimeout.c+ticks
    setTimeout.d[usetick] ||= []
    let usespot = setTimeout.d[usetick]
    usespot[usespot.length] = [()=>cb(...args),setTimeout.i,ticks] //the only difference from setTimeout
    setTimeout.r[setTimeout.i] = [usespot,usespot.length-1]
    return setTimeout.i++
}
globalThis.clearInterval = id => { //using this so == and === comparison returns false
    clearTimeout(id)
}
globalThis.setImmediate = (cb,...args) => {
    return setTimeout(cb,0,...args)
}
globalThis.clearImmediate = id => {
    clearTimeout(id)
}
globalThis.queueMicroTask = cb => {
    queueMicroTask
}
setTimeout.d = {} //dictionary of timers
setTimeout.r = {} //reverse dictionary of timers for clearing
setTimeout.q = {}
{
    let desiredTps = 20
    let changeRate = 0.001
    let desiredMs = 1000/desiredTps
    let actualMs = desiredMs
    let crComp = 1-changeRate
    let last = Date.now()
    setTimeout.t = () => {
        let now = Date.now()
        let diff = now - last
        actualMs += diff * changeRate
        last = now
        while(actualMs>=desiredMs){ //if tick is not 50ms, COMPENSATE
            last = now
            actualMs *= crComp
            //using forEach because it only iterates over non-empty slots by default
            let spot = setTimeout.d[setTimeout.c]
            if(!spot) return setTimeout.c++
            spot.forEach((arr,i)=>{
                let cb = arr[0]
                let id = arr[1]
                try{
                    cb()
                }catch(e){
                    api.broadcastMessage((arr[2]?"Interval error: ":"Timeout error: ")+e.name+": "+e.message,{color:"#ff9d87"})
                }
                if(arr[2]&&setTimeout.d[setTimeout.c][i]){
                    //if it's a interval, reschedule it for later, unless of course, it got deleted
                    let ticks = arr[2]
                    let usetick = setTimeout.c+ticks
                    setTimeout.d[usetick] ||= []
                    let usespot = setTimeout.d[usetick]
                    usespot[usespot.length] = [cb,id,ticks]
                    setTimeout.r[id]=[usespot,usespot.length-1]
                } else {
                    delete setTimeout.r[id]
                }
                delete spot[i]
            })
            delete setTimeout.d[setTimeout.c]
            setTimeout.c++
        }
    }
}
setTimeout.c = 0 //current tick
setTimeout.i = 0 //id to be used next

//usage:
//other code and callbacks
tick = () => {
    setTimeout.t()
    //rest of the code for tick
}
//other code and callbacks
```
