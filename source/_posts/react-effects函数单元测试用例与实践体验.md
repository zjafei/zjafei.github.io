---
title: react effects函数单元测试用例与实践体验
date: 2017-12-16 20:26:18
tags: ['单元测试','unit test','react','effects']
category: 'coding'
---
effects 函数是很强调业务的代码块

所以复制的业务会有很多分支代码

导致测试用例成倍的增长

虽然测试用例相同的代码可以使用  beforeAll 和 afterAll 来归纳相同代码

但是这样也使得测试代码本事的逻辑变的复杂 难以维护 难以理解

所以对 effects 函数复杂业务的拆分是解决问题的关键

把相同的业务进行抽取 做自己的的单元测试

函数只写自己特有的代码
<!--more-->
被测试代码
```javascript
import { Customer, Machine, Tenant } from 'models';
import { Error, parseResponse, request } from 'utils';
import { MessageException, NetworkException, Relocation } from 'exceptions';
import { Message, Modal } from 'antd';
 

export const api = {
    searchOneCustomerByPhone: (params) => Customer.searchOneCustomerByPhone(params),
    createCustomer: (params) => Customer.createCustomer(params),
};
export default {
    namespace: 'app',
    state: {
    },

    reducers: {
    },

    effects: {

        * catchException({ payload }, {}) {
            throw payload;
        },

        * searchCustomerFormCancel({}, { put, select }) {
            const { searchCustomerForm } = yield select(viewModelStateSelector);

            yield put({ type: 'saveIsSearchCustomerFormVisible', payload: false });
            searchCustomerForm.resetFields();
            yield put({ type: 'saveSearchCustomerFormPhone', payload: { phone: '' } });
            yield put({
                type: 'saveSearchCustomerList', payload: {
                    total_item_count: 0,
                    data_list: [],
                }
            });
        },

        * searchCustomerFormSubmit({}, { call, put, select }) {
            const { searchCustomerForm } = yield select(viewModelStateSelector);
            searchCustomerForm.validateFields();
            const wrongFields = searchCustomerForm.getFieldsError();
            if (!Object.keys(wrongFields).filter(field => wrongFields[field] !== undefined).length) {
                const { searchCustomerFormPhone } = yield select(viewModelStateSelector);

                try {
                    yield put({ type: 'saveIsSearchCustomerFormLoading', payload: true });
                    const fetchResult = yield call(request, api.searchOneCustomerByPhone(searchCustomerFormPhone.phone));
                    yield put({ type: 'saveIsSearchCustomerFormLoading', payload: false });

                    if (fetchResult.XGcbJsonCode === 0) {

                        if (fetchResult.XGcbJsonResult.customer === null) {
                            yield put({ type: 'createCustomerFormOpen' });
                        } else {
                            yield put({
                                type: 'saveSearchCustomerList', payload: {
                                    data_list: [fetchResult.XGcbJsonResult.customer],
                                    total_item_count: 1,
                                }
                            });
                        }

                    } else {
                        yield put({
                            type: 'catchException',
                            payload: new MessageException(Error.getError(fetchResult.XGcbJsonCode, {}, '无法找到客户配置信息，请重新尝试！')),
                        });
                    }
                } catch (e) {
                    if (e instanceof NetworkException) {
                        yield put({
                            type: 'catchException',
                            payload: new MessageException('网络错误，请重试！'),
                        });
                    } else {
                        throw e;
                    }
                }
            }
        }, 
    }
}
```

测试代码

```javascript
import { effects } from 'dva/saga';
import { Error, parseResponse, request } from 'utils';
import { MessageException, NetworkException, RelocationException } from 'exceptions';
import app, { viewModelStateSelector, api } from './app';

const { call, put, select } = effects;

describe('app model', () => {
    test('name space', () => {
        expect(app.namespace).toBe('app');
    });

    test('effects catchException has throw', () => {
        expect(app.effects.catchException({ payload: new MessageException('这是一个错误测试') }, {})).toThrowError();
    });

    test('effects searchCustomerFormCancel', () => {
        const resetFields = jest.fn();
        const generator = app.effects.searchCustomerFormCancel({}, { select, put });

 /***
 * 在测试代码 只关心每一行代码的情况 每一行代码需要什么 不关心代码之间的关系
 * 上一行代码值可以通过被测试行的 mock 数据来代替
 * generator 的 next 与到 yield 就停止
 * generator 的 next 得到的是 {value:上一步 yield 的返回值, done:false}
 * generator 的 next 的参数作为上一次 yield 的返回值
 * 如果没有上一步 yield 在 generator 函数内永远返回 undefined
 * yield 的返回值不是作为 generator 函数内部变量的赋值 而是近视于 return 作为 generator 函数的 next() 方法返回值的 value 值
 * generator 函数内部变量要获取 yield 的返回值 只有依赖next() 方法参数的 value 值
 *
 function* f(x) {
 for(var i = 0; true; i++) {
 var reset = yield i;//这里的 reset 还是 undefined
 console.log('reset:' + reset);//这里的 reset 就根据 next() 的传递
 if(reset===false) { i = -1; }
 }
 }
 const g = f(0);
 let x = g.next(0);
 x = g.next(x.value);
 x = g.next(x.value);
 */

 //todo 函数开始
 let next = generator.next();

  //todo 遇到第一个 yield 停止了 得到 yield 的返回值通过 next.value 等到 与期待值 select(viewModelStateSelector) 进行比较
 expect(next.value).toEqual(select(viewModelStateSelector));
        
  //todo 所以通过给next添加参数来 mock 上次 yield 的返回值
 //todo 因为在第一的 yield 是对返回值有定义的 而且对返回值有调用
 //todo 而这里的 next 获取的是 yield put({ type: 'saveIsSearchCustomerFormVisible', payload: false }); 的返回值
 next = generator.next({
            searchCustomerForm: {
                resetFields: resetFields,
            }
        });
  //todo 进行数值比较
 expect(next.value).toEqual(put({ type: 'saveIsSearchCustomerFormVisible', payload: false }));

  //todo 这里的 next 获取的是yield put({ type: 'saveSearchCustomerFormPhone', payload: { phone: '' } });的返回值
 //todo 同时我们要进行 yield select(viewModelStateSelector); 对应的 next 的输出的 mock 的函数行为测试
 //todo 这里的 next 对应的是 yield put({ type: 'saveSearchCustomerFormPhone', payload: { phone: '' } });
 next = generator.next();
  //todo mock 函数只负责行为的测试
 expect(resetFields.mock.calls.length).toBe(1);
  expect(resetFields.mock.calls[0].length).toBe(0);

  //todo 这里是对数值的比较
 expect(next.value).toEqual(put({ type: 'saveSearchCustomerFormPhone', payload: { phone: '' } }));
        next = generator.next();
        expect(next.value).toEqual(put({
            type: 'saveSearchCustomerList', payload: {
                total_item_count: 0,
                data_list: [],
            }
        }));
    });

  describe('effects searchCustomerFormSubmit', () => {//todo describe 包裹 test 反过来不测试
 test('form has error', () => {

            const mockPut = jest.fn();
            const mockCall = jest.fn();
            const validateFields = jest.fn();
            const getFieldsError = jest.fn();

            getFieldsError.mockReturnValue({});

            const generator = app.effects.searchCustomerFormSubmit({}, { mockCall, mockPut, select });

            let next = generator.next();
            expect(next.value).toEqual(select(viewModelStateSelector));
            generator.next({
                searchCustomerForm: {
                    validateFields: validateFields,
                    getFieldsError: getFieldsError,
                }
            });
            expect(validateFields.mock.calls.length).toBe(1);
            expect(getFieldsError.mock.calls.length).toBe(1);
            expect(mockCall.mock.calls.length).toBe(0);
            expect(mockPut.mock.calls.length).toBe(0);

        });

        test('fetch result has network error', () => {
            const validateFields = jest.fn();
            const getFieldsError = jest.fn();


            getFieldsError.mockReturnValue({});
            const generator = app.effects.searchCustomerFormSubmit({}, { select, call, put });

            let next = generator.next();
            expect(next.value).toEqual(select(viewModelStateSelector));
            next = generator.next({
                searchCustomerForm: {
                    validateFields: validateFields,
                    getFieldsError: getFieldsError,
                }
            });

            expect(validateFields.mock.calls.length).toBe(1);
            expect(getFieldsError.mock.calls.length).toBe(1);
            expect(next.value).toEqual(select(viewModelStateSelector));
            next = generator.next({
                searchCustomerFormPhone: {
                    phone: ''
 }
            });
            expect(next.value).toEqual(put({ type: 'saveIsSearchCustomerFormLoading', payload: true }));

            next = generator.next();
            expect(next.value).toEqual(call(request, api.searchOneCustomerByPhone('')));

            next = generator.throw(new NetworkException());
            expect(next.value).toEqual(put({
                type: 'catchException',
                payload: new MessageException('网络错误，请重试！'),
            }));
        });

        test('fetch result XGcbJsonCode === 0 customer === null', () => {
            const validateFields = jest.fn();
            const getFieldsError = jest.fn();


            getFieldsError.mockReturnValue({});
            const generator = app.effects.searchCustomerFormSubmit({}, { select, call, put });

            let next = generator.next();
            expect(next.value).toEqual(select(viewModelStateSelector));
            next = generator.next({
                searchCustomerForm: {
                    validateFields: validateFields,
                    getFieldsError: getFieldsError,
                }
            });

            expect(validateFields.mock.calls.length).toBe(1);
            expect(getFieldsError.mock.calls.length).toBe(1);
            expect(next.value).toEqual(select(viewModelStateSelector));
            next = generator.next({
                searchCustomerFormPhone: {
                    phone: ''
 }
            });
            expect(next.value).toEqual(put({ type: 'saveIsSearchCustomerFormLoading', payload: true }));

            next = generator.next();
            expect(next.value).toEqual(call(request, api.searchOneCustomerByPhone('')));

            next = generator.next({
                XGcbJsonCode: 0,
                XGcbJsonResult: {
                    customer: null
 }
            });
            expect(next.value).toEqual(put({ type: 'saveIsSearchCustomerFormLoading', payload: false }));

            next = generator.next();
            expect(next.value).toEqual(put({ type: 'createCustomerFormOpen' }));
        });

        test('fetch result XGcbJsonCode === 0 customer !== null', () => {
            const validateFields = jest.fn();
            const getFieldsError = jest.fn();


            getFieldsError.mockReturnValue({});
            const generator = app.effects.searchCustomerFormSubmit({}, { select, call, put });

            let next = generator.next();
            expect(next.value).toEqual(select(viewModelStateSelector));
            next = generator.next({
                searchCustomerForm: {
                    validateFields: validateFields,
                    getFieldsError: getFieldsError,
                }
            });

            expect(validateFields.mock.calls.length).toBe(1);
            expect(getFieldsError.mock.calls.length).toBe(1);
            expect(next.value).toEqual(select(viewModelStateSelector));
            next = generator.next({
                searchCustomerFormPhone: {
                    phone: ''
 }
            });
            expect(next.value).toEqual(put({ type: 'saveIsSearchCustomerFormLoading', payload: true }));

            next = generator.next();
            expect(next.value).toEqual(call(request, api.searchOneCustomerByPhone('')));

            next = generator.next({
                XGcbJsonCode: 0,
                XGcbJsonResult: {
                    customer: {}
                }
            });
            expect(next.value).toEqual(put({ type: 'saveIsSearchCustomerFormLoading', payload: false }));

            next = generator.next();
            expect(next.value).toEqual(put({
                type: 'saveSearchCustomerList', payload: {
                    data_list: [{}],
                    total_item_count: 1,
                }
            }));
        });

        test('fetch result XGcbJsonCode !== 0', () => {
            const validateFields = jest.fn();
            const getFieldsError = jest.fn();

            getFieldsError.mockReturnValue({});
            const generator = app.effects.searchCustomerFormSubmit({}, { select, call, put });

            let next = generator.next();
            expect(next.value).toEqual(select(viewModelStateSelector));
            next = generator.next({
                searchCustomerForm: {
                    validateFields: validateFields,
                    getFieldsError: getFieldsError,
                }
            });

            expect(validateFields.mock.calls.length).toBe(1);
            expect(getFieldsError.mock.calls.length).toBe(1);
            expect(next.value).toEqual(select(viewModelStateSelector));
            next = generator.next({
                searchCustomerFormPhone: {
                    phone: ''
 }
            });
            expect(next.value).toEqual(put({ type: 'saveIsSearchCustomerFormLoading', payload: true }));

            next = generator.next();
            expect(next.value).toEqual(call(request, api.searchOneCustomerByPhone('')));

            next = generator.next({
                XGcbJsonCode: 222,
                XGcbJsonResult: {
                    customer: null
 }
            });
            expect(next.value).toEqual(put({ type: 'saveIsSearchCustomerFormLoading', payload: false }));

            next = generator.next();
            expect(next.value).toEqual(put({
                type: 'catchException',
                payload: new MessageException('无法找到客户配置信息，请重新尝试！'),
            }));
        });
    });
});
```

测试 effects 函数主要是测试 Generator 函数每一步是否与预期的相同

相同的标准是值的相同和调取的参数是否一样

值的相同  这里强调的是结果

```javascript
expect(app.effects.searchCustomerFormSubmit({}, { select, call, put }).next().value).toEqual(put({ type: 'saveIsSearchCustomerFormLoading', payload: true }));
```

参数相同  这里强调的是行为

```javascript
expect(mockPut).toHaveBeenCalledWith({ type: 'saveIsSearchCustomerFormLoading', payload: true });
```

测试用例的覆盖最基本的是要做到每个分支都要做一个流程的测试块

```javascript
if (!Object.keys(wrongFields).filter(field => wrongFields[field] !== undefined).length)
```

对应 

```javascript
test('form has error', () => {});
```

```javascript
try
```
对应

```javascript
test('fetch result has network error', () =>{});
``` 

```javascript
if (fetchResult.XGcbJsonCode === 0) 
```
对应

```javascript
test('fetch result XGcbJsonCode === 0 customer === null', () =>{});
```

```javascript 
test('fetch result XGcbJsonCode === 0 customer !== null', () =>{});
```

```javascript
if (fetchResult.XGcbJsonResult.customer === null)
```
对应

```javascript
test('fetch result XGcbJsonCode === 0 customer === null', () =>{});
```

测试用例还要覆盖反例

```javascript
test('fetch result XGcbJsonCode !== 0', () =>{});
```

```javascript
test('fetch result XGcbJsonCode === 0 customer !== null', () =>{});
```

对于像 select, call 这样有返回数据的要在next方法中添加mock数据

```javascript
 next = generator.next({
                XGcbJsonCode: 222,
                XGcbJsonResult: {
                    customer: null
              }
});
````

mock数据要覆盖全部分支用例

```javascript
next = generator.next({
                XGcbJsonCode: 0,
                XGcbJsonResult: {
                    customer: {}
                }
});
``` 
 