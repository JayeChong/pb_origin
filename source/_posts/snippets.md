---
title: snippets
date: 2019-11-12 11:38:26
tags: [snippets]
---

#### debounce
```javascript
export function debounce (fn, delay) {
    let timer = null;

    return function () {
      if (timer) {
        clearTimeout(timer);
        timer = null;
      }

      timer = setTimeout(() => {
        fn();
        timer = null;
      }, delay);
    };
}
```

#### deepclone

```javascript
export function deepClone (obj) {
    if (typeof obj === 'object') {
        if (obj instanceof Array) {
            var newArr = [];
            for (var i = 0; i < obj.length; i++) newArr.push(deepClone(obj[i]));
            return newArr;
        } else {
            var newObj = {};
            for (var key in obj) {
                newObj[key] = deepClone(obj[key]);
            }
            return newObj;
        }
    } else {
        return obj;
    }
}
```

#### objectFilter对象根据属性过滤

```javascript
export default function objectFilter (obj) {
    if (typeof obj !== 'object') {
        return obj;
    }

    var newObj = {};

    for (var i in obj) {
        if (Object.prototype.hasOwnProperty.call(obj, i)) {
            if (obj[i] !== '' && typeof obj[i] !== 'undefined') {
                newObj[i] = obj[i];
            }
        }
    }

    return newObj;
}
```

#### 观察者模式

```javascript
let uuid = 0;
function Observer () {
    this.listeners = {};
    this.id = uuid++;
}
Observer.prototype = {
    constructor: Observer,
    sub: function (type, fn) {
        this.listeners[type] = this.listeners[type] || [];
        this.listeners[type].push(fn);
    },
    pub: function (type) {
        if (this.listeners && this.listeners[type]) {
            for (var i = 0; i < this.listeners[type].length; i++) {
                this.listeners[type][i].apply(this, [].slice.call(arguments, 1));
            }
        }
    },
    one: function (type, fn) {
        this.sub(type, function tmp (ev) {
            /* eslint-disable no-invalid-this */
            fn.call(this, ev);
            this.unbind(type, tmp);
            /* eslint-enable no-invalid-this */
        });
    },
    unbind: function (type, fn) {
        setTimeout(() => { // 如果一个事件被多个监听者监听，第一个监听者在监听的时候unbind后，这里splice去掉第1个监听，会导致pub里面的循环执行的函数减少一个，导致第二个监听handler在当前这次循环中无法执行，所以将unbind放到异步事件的宏队列执行，防止正在进行的循环的listeners会有监听handler没有执行的情况
            if (this.listeners && this.listeners[type]) {
                if (typeof fn !== 'function') {
                    delete this.listeners[type];
                } else {
                    for (var i = 0; i < this.listeners[type].length; i++) {
                        if (this.listeners[type][i] === fn) {
                            this.listeners[type].splice(i--, 1);
                        }
                    }
                }
            }
        }, 0);
    },
    unbindAll: function () {
        for (var key in this.listeners) {
            if (this.listeners.hasOwnProperty(key)) {
                delete this.listeners[key];
            }
        }
    }
};

module.exports = {
    Observer: Observer,
    observer: new Observer()
};
```

#### 手写Promise

```javascript
(function () {

  // Store setTimeout reference so promise-polyfill will be unaffected by
  // other code modifying setTimeout (like sinon.useFakeTimers())
  var setTimeoutFunc = setTimeout;

  function noop() {}

  // Polyfill for Function.prototype.bind
  function bind(fn, thisArg) {
    return function () {
      fn.apply(thisArg, arguments);
    };
  }

  function Promise(fn) {
    if (typeof this !== 'object') throw new TypeError('Promises must be constructed via new');
    if (typeof fn !== 'function') throw new TypeError('not a function');
    this._state = 0;
    this._handled = false;
    this._value = undefined;
    this._deferreds = [];

    doResolve(fn, this);
  }

  function handle(self, deferred) {
    while (self._state === 3) {
      self = self._value;
    }
    if (self._state === 0) {
      self._deferreds.push(deferred);
      return;
    }
    self._handled = true;
    Promise._immediateFn(function () {
      var cb = self._state === 1 ? deferred.onFulfilled : deferred.onRejected;
      if (cb === null) {
        (self._state === 1 ? resolve : reject)(deferred.promise, self._value);
        return;
      }
      var ret;
      try {
        ret = cb(self._value);
      } catch (e) {
        reject(deferred.promise, e);
        return;
      }
      resolve(deferred.promise, ret);
    });
  }

  function resolve(self, newValue) {
    try {
      // Promise Resolution Procedure: https://github.com/promises-aplus/promises-spec#the-promise-resolution-procedure
      if (newValue === self) throw new TypeError('A promise cannot be resolved with itself.');
      if (newValue && (typeof newValue === 'object' || typeof newValue === 'function')) {
        var then = newValue.then;
        if (newValue instanceof Promise) {
          self._state = 3;
          self._value = newValue;
          finale(self);
          return;
        } else if (typeof then === 'function') {
          doResolve(bind(then, newValue), self);
          return;
        }
      }
      self._state = 1;
      self._value = newValue;
      finale(self);
    } catch (e) {
      reject(self, e);
    }
  }

  function reject(self, newValue) {
    self._state = 2;
    self._value = newValue;
    finale(self);
  }

  function finale(self) {
    if (self._state === 2 && self._deferreds.length === 0) {
      Promise._immediateFn(function() {
        if (!self._handled) {
          Promise._unhandledRejectionFn(self._value);
        }
      });
    }

    for (var i = 0, len = self._deferreds.length; i < len; i++) {
      handle(self, self._deferreds[i]);
    }
    self._deferreds = null;
  }

  function Handler(onFulfilled, onRejected, promise) {
    this.onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : null;
    this.onRejected = typeof onRejected === 'function' ? onRejected : null;
    this.promise = promise;
  }

  /**
   * Take a potentially misbehaving resolver function and make sure
   * onFulfilled and onRejected are only called once.
   *
   * Makes no guarantees about asynchrony.
   */
  function doResolve(fn, self) {
    var done = false;
    try {
      fn(function (value) {
        if (done) return;
        done = true;
        resolve(self, value);
      }, function (reason) {
        if (done) return;
        done = true;
        reject(self, reason);
      });
    } catch (ex) {
      if (done) return;
      done = true;
      reject(self, ex);
    }
  }

  Promise.prototype['catch'] = function (onRejected) {
    return this.then(null, onRejected);
  };

  Promise.prototype.then = function (onFulfilled, onRejected) {
    var prom = new (this.constructor)(noop);

    handle(this, new Handler(onFulfilled, onRejected, prom));
    return prom;
  };

  Promise.all = function (arr) {
    var args = Array.prototype.slice.call(arr);

    return new Promise(function (resolve, reject) {
      if (args.length === 0) return resolve([]);
      var remaining = args.length;

      function res(i, val) {
        try {
          if (val && (typeof val === 'object' || typeof val === 'function')) {
            var then = val.then;
            if (typeof then === 'function') {
              then.call(val, function (val) {
                res(i, val);
              }, reject);
              return;
            }
          }
          args[i] = val;
          if (--remaining === 0) {
            resolve(args);
          }
        } catch (ex) {
          reject(ex);
        }
      }

      for (var i = 0; i < args.length; i++) {
        res(i, args[i]);
      }
    });
  };

  Promise.resolve = function (value) {
    if (value && typeof value === 'object' && value.constructor === Promise) {
      return value;
    }

    return new Promise(function (resolve) {
      resolve(value);
    });
  };

  Promise.reject = function (value) {
    return new Promise(function (resolve, reject) {
      reject(value);
    });
  };

  Promise.race = function (values) {
    return new Promise(function (resolve, reject) {
      for (var i = 0, len = values.length; i < len; i++) {
        values[i].then(resolve, reject);
      }
    });
  };

  // Use polyfill for setImmediate for performance gains
  Promise._immediateFn = (typeof setImmediate === 'function' && function (fn) { setImmediate(fn); }) ||
    function (fn) {
      setTimeoutFunc(fn, 0);
    };

  Promise._unhandledRejectionFn = function _unhandledRejectionFn(err) {
    if (typeof console !== 'undefined' && console) {
      console.warn('Possible Unhandled Promise Rejection:', err); // eslint-disable-line no-console
    }
  };

  /**
   * Set the immediate function to execute callbacks
   * @param fn {function} Function to execute
   * @deprecated
   */
  Promise._setImmediateFn = function _setImmediateFn(fn) {
    Promise._immediateFn = fn;
  };

  /**
   * Change the function to execute on unhandled rejection
   * @param {function} fn Function to execute on unhandled rejection
   * @deprecated
   */
  Promise._setUnhandledRejectionFn = function _setUnhandledRejectionFn(fn) {
    Promise._unhandledRejectionFn = fn;
  };

  module.exports = Promise;

})();
```