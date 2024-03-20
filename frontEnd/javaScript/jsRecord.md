# js
### Array.prototype.slice.call();

经常有
```js 
let str = Array.prototype.slice.call(arguments);
``` 
旨在将arguments转换为真正的数组对象str

因为arguments不是真正的数组对象，只是与数组类似，所以它并没有slice这个方法，而上面代码可以理解成是让arguments转换成一个数组对象，让arguments具有slice()方法。要是直接写arguments.slice(1)会报错。

p.s. call可以理解为将前序函数应用于()内的变量，即更换函数this，后可接参数，会依次应用于调用的函数as实参，详见javascript进阶；
slice(start, end); end不传会返回从start到数组末尾的数组。
```js
function test(c,d) {
    let a = Array.prototype.slice.call(arguments,1);
    console.log('AAA',a); // [3]
    console.log(arguments.length); // 2
}

test(2,3);
```