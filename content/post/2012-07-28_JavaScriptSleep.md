---
Categories: []
Description: ""
Tags: []
title: "JavaScript Sleep"
date: 2012-07-28T00:00:00+08:00
draft: false
---

```javascript
function sleep(millisecond)
{
	var now = new Date();
	var outTime = now.getTime() + millisecond;
	while (true)
	{
		now = new Date();
		if (now.getTime() > outTime)
		{
			return;
		}
	}
}

function Test()
{
	alert("sleep");
	sleep(3000);
	alert("continue");
}
```

```javascript
//sleep函数
var sleep = (function(){
    var queue = [],
        isFree = true;
    return function(fn, delay){
        var args = arguments,
            self = this;
        if(isFree){
            isFree = false;
            setTimeout(function(){
                fn();
                isFree = true;
                if(queue.length !== 0){
                    args.callee.apply(self, queue.shift());
                }
            }, delay);
        }else{
            queue.push(args);
        }
    }
})();

// 测试
var card = document.getElementById('a_magic_visit');
for(var i = 0; i < 100; i=i+1){
	try{
		sleep(function(){
			card.click()
		},2000); 
		sleep(function(){
			ajaxpost('magicuse_form_visit')
		},2000);
		sleep(function(){
			hideMenu()
		},2000);
	} catch(e){}
}
```
