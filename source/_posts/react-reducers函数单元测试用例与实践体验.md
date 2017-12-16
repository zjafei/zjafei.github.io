---
title: react reducers函数单元测试用例与实践体验
date: 2017-12-16 20:10:40
tags: ['单元测试','unit test','react','reducer']
---

reduces函数都是纯函数，主要是传递`payload`来修改`state`

代码都是围绕业务来编写的，这样存在的问题就是伴随业务的变化，代码也要相应的改动。那么对应的单元测试代码也要改动。而且就现在的编写思想来看业务的多少就决定了代码的数量。
<!--more-->
```javascript
reducers: {
    saveIsSearchCustomerFormLoading(state, { payload }) {
        return Object.assign({}, state, {
            isSearchCustomerFormLoading: payload,
        });
    },
    saveSearchCustomerList(state, { payload }) {
        return Object.assign({}, state, {
            searchCustomerList: {
                dataList: payload.data_list.map((customer) => {
                    return new Customer(customer);
                }),
                totalItemCount: payload.total_item_count,
            },
        });
    },
    updateSearchCustomerParams(state, { payload }) {
        return Object.assign({}, state, {
            searchCustomerParams: Object.assign({}, state.searchCustomerParams, payload),
        });
    },


    saveCreateCustomerForm(state, { payload }) {
        return Object.assign({}, state, {
            createCustomerForm: payload,
        });
    },
    saveIsCreateCustomerFormVisible(state, { payload }) {
        return Object.assign({}, state, {
            isCreateCustomerFormVisible: payload,
        });
    },
    saveIsCreateCustomerFormLoading(state, { payload }) {
        return Object.assign({}, state, {
            isCreateCustomerFormLoading: payload,
        });
    },
    saveCreateCustomerFormData(state, { payload }) {
        return Object.assign({}, state, {
            createCustomerFormData: payload,
        });
    },
    updateCreateCustomerFormData(state, { payload }) {
        return Object.assign({}, state, {
            createCustomerFormData: Object.assign({}, state.createCustomerFormData, payload),
        });
    },
},
```

但是观察代码后发现，这些函数主要处理的业务都是对state属性的赋值和扩展，而且reduces又是纯函数，所以对函数的抽象就很有必要，也相对简单。

```javascript
reducers: {
 saveStateProperty(state, { payload }) {
        const { name, value } = payload;
        if (state[name] !== undefined) {
            return Object.assign({}, state, {
                [name]: value,
            });
        } else {
            return state
        }
    },

 updateStateProperty(state, { payload }) {
        const { name, value } = payload;
        if (state[name] !== undefined) {
            return Object.assign({}, state, {
                [name]: Object.assign({}, state[name], value),
            });
        } else {
            return state
        }
    },
}
```

主要变为两个函数 saveStateProperty 负责对某个属性的赋值；updateStateProperty：负责对某个属性的扩展；（其实扩展方法就可以实现赋值的方法，但是考虑函数的功能的单纯和参数的简单，还是分为了两个函数）

基于上面的实现那么对于reducers测试就不会随着业务的增长而增长。

测试用例如下

```javascript
describe('reducers', () => {
    test('saveStateProperty has property as payload\'s name', () => {
        expect(app.reducers.saveStateProperty({ test: 'no' }, {
            payload: {
                name: 'test',
                value: 'ok'
 }
        })).toEqual({ test: 'ok' });
    });

    test('saveStateProperty has property as payload\'s name', () => {
        expect(app.reducers.saveStateProperty({ test: 'no' }, {
            payload: {
                name: 'test',
                value: 'ok'
 }
        })).not.toEqual({ test: 'no' });
    });

    test('saveStateProperty with out property as payload\'s name', () => {
        expect(app.reducers.saveStateProperty({}, { payload: { name: 'test', value: 'ok' } })).toEqual({});
    });

    test('saveStateProperty with out property as payload\'s name', () => {
        expect(app.reducers.saveStateProperty({}, {
            payload: {
                name: 'test',
                value: 'ok'
 }
        })).not.toEqual({ test: 'ok' });
    });
});
```

其实effects部分的函数也存在业务代码过多的问题，而且情况更严重。但是副作用函数的不确定性本身就会带来很多的业务分支，所以现在没有想出什么好的办法。
但是基于控件的使用倒是有一定的规律可行。现在的控件基本就是form的验证处理，modal的显示隐藏，list的处理和筛选，还有详情页的字段显示。可以把这些常用组件有规律的事件做一些封装。