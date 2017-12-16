---
title: react model函数单元测试用例与实践体验
date: 2017-12-16 20:42:59
tags: ['单元测试','unit test','react','model']
---
构造函数的主要作用就是对输入的对象进行类别的区分。以及暴露出一个修饰过的fetch函数。

源代码：

<!--more-->
```javascript
import { camelCase } from 'utils';
import fetch from 'dva/fetch';
 
export default class CrmModel {
    constructor(args) {
        if (typeof args === 'object' && args !== null) {
            Object.keys(args).forEach((key) => {
                this[key] = args[key];
            });
        }
    }
}
 
export const addReactHeader = (options = { headers: new Headers() }) => {
    if (options.headers instanceof Headers === false) {
        options.headers = new Headers();
    }
    options.headers.append('X-React-Request', 1);
    return options;
};
 
export const modifyFetch = (fun1, fun2) => {
    if(typeof fun1 === 'function' && typeof fun2 === 'function'){
        return function () {
            const [url, options, ...args] = arguments;
            return fun1(url, fun2(options));
        }
    }
};
 
export const crmFetch = modifyFetch(fetch, addReactHeader);
```

测试代码

```javascript
import CrmModel, { addReactHeader, modifyFetch, crmFetch } from './CrmModel.js';
import fetch from 'dva/fetch';
 
describe('CrmModel', () => {
    describe('constructor', () => {
        test('constructor no arg', () => {
            expect(new CrmModel()).toEqual({});
        });
 
        test('constructor arg type of object', () => {
            const cm = new CrmModel({
                test: '0'
            });
            expect(cm).toEqual({ test: '0' });
            expect(cm).not.toEqual({ test: '1' });
        });
    });
 
    describe('add React Header', () => {
        test('no arg', () => {
            expect(addReactHeader()).toEqual({ "headers": { "_headers": { "x-react-request": [1] } } });
        });
 
        test('has arg no headers', () => {
            expect(addReactHeader({ name: 'eric' })).toEqual({
                "name": "eric",
                "headers": { "_headers": { "x-react-request": [1] } }
            });
        });
 
        test('has arg has headers but headers is not instance of Headers', () => {
            expect(addReactHeader({ name: 'eric', headers: { 'test': 1 } })).toEqual({
                "name": "eric",
                "headers": { "_headers": { "x-react-request": [1] } }
            });
        });
 
        // test('has arg has headers and headers is instance of Headers', () => {
        //     expect(addReactHeader({ name: 'eric' })).toEqual({//todo 无法获取 Headers
        //         "name": "eric",
        //     });
        // });
    });
 
    describe('modify Fetch', () => {
        test('no args ', () => {
            expect(modifyFetch()).toEqual(undefined);
        });
 
        test('has args but type not function', () => {
            expect(modifyFetch('a', 'b')).toEqual(undefined);
        });
 
        test('has args but type not function', () => {
            expect(modifyFetch(function () {
            }, 'b')).toEqual(undefined);
        });
 
        test('has args but type not function', () => {
            expect(modifyFetch('a', function () {
            },)).toEqual(undefined);
        });
 
        describe('has args and type is function', () => {
            test('return is function and args has be call', () => {
                const mockFun0 = jest.fn();
                const mockFun1 = jest.fn();
                const fun = modifyFetch(mockFun0, mockFun1);
 
                mockFun1.mockReturnValue(5);
                fun('a', 'b');
 
                expect(typeof fun).toBe('function');
                expect(mockFun0.mock.calls.length).toBe(1);
                expect(mockFun0.mock.calls[0][0]).toBe('a');
                expect(mockFun0.mock.calls[0][1]).toEqual(5);
                expect(mockFun1.mock.calls.length).toBe(1);
                expect(mockFun1.mock.calls[0][0]).toBe('b');
            });
 
            test('return is function and args has be call', () => {
                const fun0 = function (x, y) {
                    return x + y;
                };
                const fun1 = function (y) {
                    return y * y;
                };
                const fun = modifyFetch(fun0, fun1);
                expect(fun(2, 3)).toBe(11);
            });
        });
    });
 
    test('return promise', () => {
        expect(crmFetch() instanceof Promise).toBe(true);
    });
 
});
```