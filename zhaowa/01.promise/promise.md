# Promise规范及应用


## Promise States
- pending 初始状态，可改变
  - resolve -> fulfilled
  - reject -> rejected
- fulfilled 最终态, 不可变
  - 必须拥有value
- rejected  最终态，不可变
  - 必须拥有一个reason

## then
> Promise 提供一个then方法，用来访问最终的结果

- `promise.then(onFulfilled, onRejected)`
  - `onFulfilled | onRejected`必须是函数，如果不是，应该被忽略
    - 应该是微任务 通过`queueMicrotask` 来实现
  - `states = fulfilled` 时 应该调用 `onFulfilled`,参数是 value, 且只调用一次
  - `states = rejected` 时 应该调用 `onRejected`,参数是 reason, 且只调用一次
- `then`方法可被多次调用
  - 所有`onFulfilled | onRejected`都要按`then`的顺序执行(此时需要数组来存储回调)
- `then` 返回一个


### 实现一个Promise
```javascript
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise{
  FULFILLED_CALLBACK_LIST = []
  REJECTED_CALLBACK_LIST = []
  _status = PENDING

  constructor(fn){
    this.status = PENDING
    this.value = null
    this.reason = null

    try {
      fn(this.resolve.bind(this), this.reject.bind(this))
    } catch (error) {
      this.reject(error)
    }
    
  }

  get status(){
    return this._status
  }

  set status(newStatus){
    this._status = newStatus
    switch (newStatus) {
      case FULFILLED:
        this.FULFILLED_CALLBACK_LIST.forEach(callback =>{
          callback(this.value)
        })
        break;
      case REJECTED:
        this.REJECTED_CALLBACK_LIST.forEach(callback =>{
          callback(this.reason)
        })
        break
      default:
        break;
    }
  }

  resolve(value){
    if(this.status == PENDING){
      this.value = value
      this.status = FULFILLED
    }
  }
  reject(reason){
    if(this.status == PENDING){
      this.reason = reason
      this.status = REJECTED
    }
  }
  isFunction(param){
    return typeof param === 'function'
  }
  then(onFulfilled, onRejected){
    const realOnFulfilled = this.isFunction(onFulfilled) ? onFulfilled : (value) =>{
      return value
    }
    const realOnRejected = this.isFunction(onRejected) ? onRejected : (reason) =>{
      return reason
    }

    const promise2 = new MyPromise((resolve, reject) => {
      const fulfilledMicrotack = () =>{
        queueMicrotask(() => {
          try {
            resolve(realOnFulfilled())
          } catch (error) {
            reject(error)
          }
        })
      }
      const rejectedMicrotack = () =>{
        queueMicrotask(() => {
          try {
            resolve(realOnRejected())
          } catch (error) {
            reject(error)
          }
        })
      }

      switch(this.status){
        case FULFILLED:
          fulfilledMicrotack()
          break
        case REJECTED:
          rejectedMicrotack()
          break
        case PENDING:
          this.FULFILLED_CALLBACK_LIST.push(fulfilledMicrotack)
          this.REJECTED_CALLBACK_LIST.push(rejectedMicrotack)
      }
    })

    return promise2
  }

  catch(onRejected){
    return this.then(null, onRejected)
  }

  finally(callback){
    return this.then((value) => {
      return MyPromise.resolve(callback()).then(() => value)
    }, (err) => {
      return MyPromise.resolve(callback()).then(() => {throw err})
    })
  }

  static resolve(value){
    if(value instanceof MyPromise){
      return value
    }
    return new MyPromise((resolve) => {
      resolve(value)
    })
  }

  static reject(reason){
    return new MyPromise((resolve,reject) =>{
      reject(reason)
    })
  }

  static race(list){
    return new MyPromise((resolve, reject) => {
      const length = list.length
      if(length === 0) return resolve()
      list.forEach(item => {
        MyPromise.resolve(item).then((value) =>{
          return resolve(value)
        },(reason) => {
          return reject(reason)
        })
      })
    })
  }

  static all(list){
    return new MyPromise((resolve, reject) => {
      let res = []
      let count = 0
      for(let i = 0; i < list.length; i++){
        MyPromise.resolve(list[i]).then((value) =>{
          res[i] = value
          count++
          if(count === list.length){
            resolve(res)
          }
        }).catch((reason) => {
          reject(reason)
        })
      }
    })
  }
}

```



