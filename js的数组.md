```
>>> x = new Array(3)
[undefined, undefined, undefined]
>>> y = [undefined, undefined, undefined]
[undefined, undefined, undefined]

>>> x.constructor == y.constructor
true

>>> x.map(function(){ return 0; })
[undefined, undefined, undefined]
>>> y.map(function(){ return 0; })
[0, 0, 0]
```
上述结果的原因是啥呢？[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)上有详细描述，当然es6规范也有详细规范接下来，我来翻译一下：
1. map方法会为数组中的每一个元素依次按序调用一次cb,并且以返回的结果构造一个新的数组，cb仅仅会为那些被赋过值(赋的值是undefined也可以)的数组下标调用, 那么上面的问题就很好解释了，new Array(3)仅仅是定义了一个长度为3的数组，而没有为数组的任何一个下标位置赋过值，所以根本就没有调用过cb，当然map后就不会有值了, map的cb是不会为那些从未赋值过得index调用一遍的或者是被deleted,又或者是从未被赋值过得
2. map方法还有第二个参数，thisArg，传递个cb的this值
3. map是不会更改原数组的
4. 数组的范围处理（即数组有多少元素）是在map的cb调用之前进行的，那就意味着，在cb调用时，对数组添加元素或是删除元素都不会对cb可见，但是若是改变其中的值到是可见的
5. 根据规范定义算法，若是map在一个稀疏数组上调用，那么返回的数组也是保持相同的稀疏结构

这是MDN种给的一个map的实现方式
```
// Production steps of ECMA-262, Edition 5, 15.4.4.19
// Reference: http://es5.github.io/#x15.4.4.19
if (!Array.prototype.map) {

  Array.prototype.map = function(callback/*, thisArg*/) {

    var T, A, k;

    if (this == null) {
      throw new TypeError('this is null or not defined');
    }

    // 1. Let O be the result of calling ToObject passing the |this| 
    //    value as the argument.
    var O = Object(this);

    // 2. Let lenValue be the result of calling the Get internal 
    //    method of O with the argument "length".
    // 3. Let len be ToUint32(lenValue).
    var len = O.length >>> 0;

    // 4. If IsCallable(callback) is false, throw a TypeError exception.
    // See: http://es5.github.com/#x9.11
    if (typeof callback !== 'function') {
      throw new TypeError(callback + ' is not a function');
    }

    // 5. If thisArg was supplied, let T be thisArg; else let T be undefined.
    if (arguments.length > 1) {
      T = arguments[1];
    }

    // 6. Let A be a new array created as if by the expression new Array(len) 
    //    where Array is the standard built-in constructor with that name and 
    //    len is the value of len.
    A = new Array(len);

    // 7. Let k be 0
    k = 0;

    // 8. Repeat, while k < len
    while (k < len) {

      var kValue, mappedValue;

      // a. Let Pk be ToString(k).
      //   This is implicit for LHS operands of the in operator
      // b. Let kPresent be the result of calling the HasProperty internal 
      //    method of O with argument Pk.
      //   This step can be combined with c
      // c. If kPresent is true, then
      if (k in O) {

        // i. Let kValue be the result of calling the Get internal 
        //    method of O with argument Pk.
        kValue = O[k];

        // ii. Let mappedValue be the result of calling the Call internal 
        //     method of callback with T as the this value and argument 
        //     list containing kValue, k, and O.
        mappedValue = callback.call(T, kValue, k, O);

        // iii. Call the DefineOwnProperty internal method of A with arguments
        // Pk, Property Descriptor
        // { Value: mappedValue,
        //   Writable: true,
        //   Enumerable: true,
        //   Configurable: true },
        // and false.

        // In browsers that support Object.defineProperty, use the following:
        // Object.defineProperty(A, k, {
        //   value: mappedValue,
        //   writable: true,
        //   enumerable: true,
        //   configurable: true
        // });

        // For best browser support, use the following:
        A[k] = mappedValue;
      }
      // d. Increase k by 1.
      k++;
    }

    // 9. return A
    return A;
  };
}
```

这是lodash中的map的实现
```
/**
 * Creates an array of values by running each element of `array` thru `iteratee`.
 * The iteratee is invoked with three arguments: (value, index, array).
 *
 * @since 5.0.0
 * @category Array
 * @param {Array} array The array to iterate over.
 * @param {Function} iteratee The function invoked per iteration.
 * @returns {Array} Returns the new mapped array.
 * @example
 *
 * function square(n) {
 *   return n * n
 * }
 *
 * map([4, 8], square)
 * // => [16, 64]
 */
function map(array, iteratee) {
  let index = -1
  const length = array == null ? 0 : array.length
  const result = new Array(length)

  while (++index < length) {
    result[index] = iteratee(array[index], index, array)
  }
  return result
}

export default map
```