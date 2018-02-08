# Programming In 2018
## 这不是教程，只是整理。错漏之处望见谅。

***
## 不可变、无状态与状态管理
- [Go语言map是怎么比较key是否存在的](https://www.zhihu.com/question/27708751/answer/236601975)
- [下一代状态管理工具 immer 简介及源码解析](https://zhuanlan.zhihu.com/p/33507866)

可变：传引用，传指针

原值不可变：传值，传结构体

语言层面的不可变：每一次修改返回新的拷贝

无变量状态管理：参数锁存(根据传递参数，维护状态机)，STM(Software transaction memory)，数据库（这个不用解释）

class:
``` javascript
    class Page extends Component {
        state = {
            project: {
                info: '',
                user: [],
            }
        }
        onUserPull = users => this.setState({
            project: {
                ...this.state.project,
                user: [
                    ...this.state.project.user,
                    users,
                ],
            },
        });
        render() {
            return (
                <View>{Object.entries(this.state.user).map((name, users) => (
                    <List>
                        <ListHead>{name}</ListHead>
                        <ListBody>{users.map({ name, phone } => (
                            <ListItem>
                                <Text>Name: {namme}</Text>
                                <Text>Phone: {namme}</Text>
                            </ListItem>
                        ))}</ListBody>
                        <ListFoot>
                            <Button onClick={() =>
                                pullUser().then(this.onUserPull)
                            }>More</Button>
                        </ListFoot>
                    </List>
                ))}</View>
            );
        }
    }
    export default Page;
```

function:
``` javascript
    const store = {
        project: {
            info: '',
            user: [],
        },
    };
    const reduce = {
        project: {
            user: {
                join: (state, usesr) => [...state, users],
            },
        }
    };
    const Page = ({ project: { info, user } }, { dispatch }) => (
        // ...
        <Button onClick={() =>
            pullUser().then(users => dispatch({ type: 'project.user.join', payload: users }))
        }></Button>
        // ...
    );
    export connect({ project } => { project })(Page);
```

trigger.:
``` javascript
`
    Q      !
    |      Q
    o      |
    [&]-|  o
    | |_|_[&]
    |   |_| |
    R       S
`
```


***
## 用Steam(流)规避循环
``` java
// ### 循环：
{
    // - [each](file:///E:\MyProject\Project\sd\code\server\base\world\src\com\pwrd\yt\worldsrv\rank\RankGlobalService.java#L92)
    stream.forEach();
    // - [each-map](file:///E:\MyProject\Project\sd\code\server\base\world\src\com\pwrd\yt\worldsrv\rank\AbsGroupRankObject.java#303)
    stream().map().forEach();
    // - [each-filter](file:///E:\MyProject\Project\sd\code\server\base\world\src\com\pwrd\yt\worldsrv\rank\AbsGroupRankObject.java#149)
    stream().filter(shouldRank).forEach();
}

// ### 查找：
{
    // - 查找位置：[filter-count](file:///E:\MyProject\Project\sd\code\server\base\world\src\com\pwrd\yt\worldsrv\singleTower\SingleTowerManager.java#L541)
    stream().filter().count();
    // - 查找元素：[filter-find](file:///E:\MyProject\Project\sd\code\server\base\world\src\com\pwrd\yt\worldsrv\rank\AbsGroupRankObject.java#L357)
    stream().filter().findAny().orElse();
    // - 元素&位置：[filter-peek-find](file:///E:\MyProject\Project\sd\code\server\base\world\src\com\pwrd\yt\worldsrv\rank\AbsGroupRankObject.java#L283)
    AtomicInteger index = new AtomicInteger();
    E element = stream().peek().filter().findFirst().orElse();
}

// ### 安全迭代：
{
    // - [range-else](file:///E:\MyProject\Project\sd\code\server\base\bridge\src\com\pwrd\yt\bridge\bridgeSeekHegemony\BridgeSeekHegemonyService.java#1990)
    IntStream.range(0, limit).mapToObj(i -> buildIterObj).map(doSth).peek(doSth).filter(Objects::nonNull).findFirst().orElseThrow(() -> new Exception());
}

// ### 切片采样：
{
    // - [skip-limit](file:///E:\MyProject\Project\sd\code\server\base\world\src\com\pwrd\yt\worldsrv\rank\AbsGroupRankObject.java#232)
    stream().skip().limit().forEach();
    // - sort&reverse:
    stream().sorted(Comparator.reverseOrder()).forEach();
    // - iterate:
    List<String> users = Arrays.asList("Alice", "girl", "bob", "boy");
    Set<String> names = IntStream.iterate(0, i -> 1 + i)
            .mapToObj(users::get)
            .collect(Collectors.toSet());
    Map<String, String> user = IntStream.iterate(0, i -> 2 + i)
            .mapToObj(i -> new AbstractMap.SimpleImmutableEntry<>(users.get(i), users.get(1 + i)))
            .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
}

// ### 注意！
// - 保证数据的不可变
// - 谓词提取
```


***
## 条件与分支
### 流程控制vs表达式
重新审视三目：保证了值的确定与不变
``` javascript
    let variable;
    if (condition) {
        variable = state0;
    } else {
        variable = state1;
    }
    doSth(variable);
```
``` javascript
    const variable = condition ? state0 : state1;
    doSth(variable);
```

大量的 `if-elif-else` 造成逻辑的分歧
[轴映射](https://www.zhihu.com/question/266058426/answer/302555580)

`switch` 也同理，可以用哈希表映射成表达式
``` javascript
    const variable = ({ // 事实上，这种简单的状态机在动态语言非常常见，以至于有些语言不设switch语句。
        case0: state0,
        case1: state1,
        get case2() { // 注意这个case2，通过getter访问器而不是简单值，提供了延迟执行能力
            doSth();
            return state2;
        },
    })[condition];
```
基于这个原理，一个变量的是非常确定的，它不是因某个条件变成某个值，而是对一种状态就有一个值。

但事实上并不是说if-else, switch结构本身有错，而是像 `goto` 语句一样的，早期Basic这样面向过程的语言残留的缺陷。
比如在一些函数式语言中，if-else事实上是长这样的：
``` lisp
    ; (set variable (if condition state0 (doSth state1))) ; 这是inline写法
    (set variable (
        if condition (
            state0
        ) (
            doSth
            state1
        )
    ))
```
``` elixir
    # variable = if condition, do: state0, else: state1 # 这是inline写法
    variable = if condition do
        state0
    else
        doSth
        state1
    end
```
``` es6
    // 处于草案阶段但很多项目已经直接用上的JS的do-if
    const variable = do {
        if condition {
            state0;
        } else {
            state1;
        }
    };
```
上面两例都是表达式，它们根据条件真假，执行if或else作用域块内表达式，并返回最后的值。

另外， `if` 还有另一种常见场景，特殊逻辑处理，比如：
``` javascript
    if (needDoSth) {
        doSth();
    }
    doElse();
```
但如果所用的语言支持非布尔值的逻辑短路运算，其实这样更好：
``` javascript
    needDoSth && doSth();
    doElse();
```
同理还有：
``` javascript
    hasDone || doSth();
    doElse();
```
这样的好处也是将过程的分叉变成了普通的表达式求值问题。看下面的例子：
``` javascript
{
    // T_T
    if (user == null) {
        return;
    }
    show(user);

    // ^_^
    user && show(user);
}
{
    // T_T
    if (user == null) {
        show("not found");
        return;
    }
    show(user);

    // ^_^
    show(user || "not found");
}
{
    // T_T
    let display = name;
    if (!user.name) {
        display = user.id;
    }
    show(display);

    // ^_^
    show(user.name || user.id);
}
```
珍爱眼睛，谨防老花。

残念，Java不支持支持非布尔的逻辑短路。但作为代替，Java提供了以下方法：
``` Java
    user.ifPresent(show);
    show(user.orElse("not found"));
    show(user.getOrDefault("name", user.get("id")));
```
但这可能是好事，避免了隐式的真值判断的混淆：
``` javascript
[] == false
![] == false
[] == ![]
false || 'pass' // 'pass'
[] || 'pass' // []
// fuck.js
```

### 分支收束
[logic gather](E:\MyProject\Project\sd\code\server\weekly\world\src\com\pwrd\yt\worldsrv\activity\bridgeSeekHegemony\BridgeSeekHegemonyManager.java#L567)
一些情况，逻辑实际是收束的，比如：123轮到Alice，456轮到Bob，周日（0）两人一起值日。
``` javascript
    // bad
    if (tody == 0)  {
        alice.works(...args);
        bob.works(...args);
    } else if (tody <= 3) {
        alice.works(...args);
    } else {
        bob.works(...args);
    }

    // better
    const works = do {
        if (tody == 0)  {
            (...args) => (alice.works(...args), bob.works(...args));
        } else if (tody <= 3) {
            alice.works;
        } else {
            bob.works;
        }
    };
    // 注意这里 `alice` 与 `bob` 并没有关系，没必要Ta们有相同父类抽象之类的，只要能获取a或b或a&b的绑定方法。
    works(...args);
```
当然，这里坐标轴也很明显，换成轴映射就更清晰了：
``` javascript
    const whoes = [
        (0, [alice, bob]),
        (3, alice),
        (6, bob),
    ].find(([point, whoes]) => point <= today)[1];
    combine(whoes).works(...args); // 关于 `combine` 后面解释。
```
或者换个角度，我们知道三种case对应不同的执行者，那么除了坐标轴外，我们也可以表达成映射表：
``` javascript
    const mapper = [[alice, bob], [alice], [bob]]; // { 0: [alice, bob], 1: alice, 2: bob }
    const whoes = mapper['0111222'[tody]]; // JS是弱类型
    combine(whoes).works(...args);
```
优点是灵活，比如改成没规律的，026-A，13-B，45-AB之类的。


***
## 结构与解构
### 基本结构
``` javascript
{ key: 'value' } // dict, object, table, hashmap
['value0', 'value1', 'value2'] // list, array
```
``` python
{ unique0, unique1, unique0 } # set, dict的特例, { unique: true }
('value0', 'value1', 'value2') # tuple, list的特例, 不可变的list
```

### 列表处理
一个重要概念：map
``` javascript
[1, 2, 3].map(i => String.fromCharCode(i + 64)); // ["A", "B", "C"]
```

另一个重要概念：reduce
``` javascript
    // count
    [1, 2, 3].reduce(c => 1 + c, 0); // = 3

    // sum
    [1, 2, 3].reduce((s, i) => s + i); // = 6

    // dict
    [['one', 1], ['two', 2], ['three', 3]].reduce((dict, [name, value]) => (dict[name] = value, dict), {}); // {one: 1, two: 2, three: 3}

    // flat
    [[0, 1, 2], [3, 4, 5]].reduce((r, t) => [...r, ...t]); // [0, 1, 2, 3, 4, 5]
    [[0, 1, 2], [3, 4, 5]].join().split(',') // 另一种实现，但这只是JS的黑科技，不推荐

    // group
    const arr = [0, 1, 2, 3, 4, 5];
    new Array(2).fill().map((_, i) => new Array(3).fill().map((_, ii) => arr[3 * i + ii])); // [[0, 1, 2], [3, 4, 5]]

    // zip
    const rows = [['one', 'two'], [1, 2]];
    rows[0].map((_, i) => rows.map(row => row[i])); // ['one', 1, 'two', 2]
    // unzip
    const arr = ['one', 1, 'two', 2];
    new Array(2).fill().map((_, i) => new Array(2).fill().map((_, ii) => arr[i + 2 * ii]))
```

通过map-reduce（以及filter、slice等各种列表处理方法），我们可以灵活的处理列表结构。

另外，一些真*高级语言里，提供了直接的列表推导支持：
``` javascript
    [1, ...[2, 3]]; // [1, 2, 3]
```
``` elixir
    [1 | [2, 3]]
    [1, 2] ++ [3, 4] # [1, 2, 3, 4]
    0..9
```
``` python
    [1, *[2, 3]]
    [1, 2] + [3, 4]
    # range(10)
    [map_with(item) for item in items if filter(item)] # 列表生成式，这不是控制流程，是表达式，items.filter().map()
```

### `Lisp` (List Processing) 告诉我们：一切皆列表
> 即：先考虑转化为列表，再进行推导。
``` java
    // - [HashMap::entrySet](file:///E:\MyProject\Project\sd\code\server\base\world\src\com\pwrd\yt\worldsrv\rank\AbsGroupRankObject.java#L247)
    hashmap.entrySet().stream()//.map().filter()...
```
``` javascript
    Object.entries(object)//.map().filter()...
```
``` python
    # !notice 在Python里，我们是可以直接对`dict`进行推导的
    student = { 'Alice': 60, 'Bob': 59 }
    [name for name in student if student[name] < 60]
    # 当然我们仍然可以强制得到一个`list`(事实上是`set`)然后进行推导
    dict.items()
```

### 并发、分布式 `map-reduce` 、惰性求值
``` java
    parallelStream(); // 并发流
    IntStream.generate(); // 流生成器
```
``` python
    range(100**100); # 生成范围
    (i for i in range(100**100)) # 生成器
```

### 解构
``` javascript
const [a, b, ...c] = [1, 2, 3]; // a: 1, b: 2, c: [3]
const { name, ...extra } = { name: 'Alice', age: 18 }; // name: 'Alice', extra: { age: 18 }
```
``` elixir
[a, b | c] = [1, 2, 3] # 这其实弱化的模式匹配，尝试：[a, a | c] = [1, 2, 3]
[{:name, name}, extra] = [name: 'Alice', age: 18] # %{name: name} = %{name: 'Alice'}
```
``` python
a, b, *extra = 1, 2, 3 # [a, b, *ext] = [1, 2, 3]
```

### 重构
``` javascript
    const serial = ({id, name, age}) => [id, name, age];
    const deserial = ([id, name, age]) => ({ id, name, age });
    serial({ id: 0, name: 'Alice', age: 18 }); // [0, "Alice", 18]
    deserial([ 0, 'Alice', 18 ]); // {id: 0, name: "Alice", age: 18}
```

### 善用结构变化
``` javascript
const userFlag = {
    isManager: false,
    isAdult: true,
    verified: {
        phone: true,
        email: false,
    },
};
const toBit = ({ isManager, isAdult, verified: { phone, email } }) =>
    [isManager, isAdult, phone, email].reverse().reduce((r, t, i) => t ? r | 2 ** i : r, 0);
const toFlag = bit => (([isManager, isAdult, phone, email]) => ({ isManager, isAdult, verified: { phone, email } }))(
    Array.from(bit.toString(2)).map(c => parseInt(c)));
```

### 流|集合 的再思考
> 可能放在最后有时间再讲解比较好？
`combine` 也是一种解决方法，比如红极一时的 `jQuery`：
``` javascript
    const nodes = (div => (div.innerHTML = '<span>a</span><span>b</span><span>c</span>', div.querySelectorAll('span')))(document.createElement('div'));
    $(nodes[0]).empty();
    $(nodes).empty();
```
实现起来也简单：
``` javascript
    const combine = arr => new Proxy(arr, {
        get: (target, key) => {
            const fns = Array.prototype.reduce.call(
                target, (fns, ele) => fns && (ele[key] instanceof Function ? [...fns, ele[key].bind(ele)] : false), [],
            );
            if (!fns) {
                return Array.prototype.map.call(target, ele => ele[key])
            }
            return (...args) => fns.map(fn => fn(...args));
        },
        set: (target, key, value) => Array.prototype.forEach.call(target, ele => ele[key] = value),
    });
```

***
## 模式匹配
### 面向Logic
在[SWISH](https://swish.swi-prolog.org/)尝试：
``` prolog
match(OK) :- ok = OK.
```

### 二进制模式匹配
``` erlang
<< IsManager:1, IsAdult:1, Verified:2 >> = << 6:4 >>.
```
``` elixir
<< isManager::1, isAdult::1, verified::2 >> = << 6::4 >>
```


### 解构
参见上一节

### 面向人的重载
``` elixir
handle = fn
    200, result -> "ok, " <> result
    code, result when code < 500 -> "#{code}, #{result}"
    _, result -> "service inner error! #{result}"
end

handle.(200, "pass")
handle.(404, "not found!")
handle.(502, "Bad Gateway.")
```
``` javascript
const mapper = [
    [c => c == 200, (c, r) => `ok, ${r}`],
    [c => c < 500,  (c, r) => `${c}, ${r}`],
    [c => true,     (c, r) => `service inner error! ${r}`],
];
const handle = (code, result) => mapper.find(([predicate, action]) => predicate(code))[1](code, result);
```
处于草案阶段的JS的 [pattern-match](https://github.com/tc39/proposal-pattern-matching#ecmascript-pattern-matching-syntax)


***
## 链式
### 对象的链式调用
上面其实已经出现过多次了。本质是静态工厂模式，方法调用后返回接受者本身。更典型的是Promise模型：
``` javascript
Promise.resolve({})
    .then(o => (o.name = 'Alice', o))
    .then(o => (o.age = 18, o))
    .then(o => fetch('').then(data => o.data = data))
    .catch(console.err);
```
这在异步环境下优势十分明显，具体后面讲解。

### 函数的compose
以下条件：
1. 根据 `ordinal` 可以拿到 `period` .
2. 根据 `period` 可以拿到 `voList` .
3. 根据 `voList` 可以拿到 `vo` .
4. 根据 `vo` 可以拿到 `prop` .
以下需求：
1. 已知 `ordinal` ，分别求 `voList`, `vo`, `prop` 。
2. 求下一目标前，可能会对上一目标追加推导，且各种场景不同。

不考虑 `2` 的前提下，可能会写出这样的代码：
``` javascript
    const periodFromOrdinal = ordinal => period;
    const voListFromPeriod = period => voList;
    const voFromList = voList => vo;
    const propOfVO = vo => prop;

    const queryProp = ordinal => propOfVO(voFromList(voListFromPeriod(periodFromOrdinal(ordibal))));
```
有经验的程序员一般都不会这么写，因为这样的方法调用一层套一层，而且是读起来反向的，得从里向外读。

另一个重要的原因则是不好扩展，比如现在要加入需求 `2` 。所以一般我们宁可多用几个局部变量，一般实现：
``` javascript
    const queryVOList = ordinal => {
        const period = periodFromOrdinal(ordinal);
        if (!filter1(period)) {
            return;
        }
        const list = voListFromPeriod(period);
        doSth1(list);
        return list;
    };
    const queryVO = ordinal => {
        const period = periodFromOrdinal(ordinal);
        if (!filter2(period)) {
            return;
        }
        let list = voListFromPeriod(period);
        list = map1(list);
        const vo = voFromList(list);
        if (!filter3(vo)) {
            return;
        }
        doSth2(vo);
        return vo;
    };
```
但试想这样的 `filter`, `doSth`, `map` 等不断增加下去，各种组合下去。。。

这种需求其实普遍存在，比如，数据库的查询。那么数据库查询语句，SQL，是怎么解决这个问题的呢？答案就是链式与组合。

1. [Java的Function的compose与then](file:///E:\MyProject\Project\sd\code\server\base\bridge\src\com\pwrd\yt\bridge\bridgeSeekHegemony\BridgeSeekHegemonyService.java#1982)
``` Java
    // Function<From, To> chainTFromF() { return from -> to; }
    Function<Integer, Period> chainPeriodFromOrdinal();
    Function<Period, VOList> chainVOListFromPeriod();
    Function<VOList, VO> chainVOFromVOList();
    Function<VO, Prop> chainPropFromVO();
    Period filterWithPeriodA(Period period);
    Period filterWithPeriodB(Period period);
    VOList mapWithVOListA(VOList list);
    VO doWithVOA(VO vo);
    ...
    Prop queryProp_FilterPeriodB_DoWithVOListC_MapWithVOD(Integer ordinal) {
        Function<Integer, Prop> propChain = chainPeriodFromOrdinal()
            .andThen(chainVOListFromPeriod())
            .andThen(filterPeriodB)
            .andThen(chainVOFromVOList())
            .andThen(doWithVOList)
            .andThen(chainPropFromVO())
            .andThen(mapWithVO);
        return propChain.apply(ordinal);
    }
```

2. 一切皆列表，函数列表compose、pipe
``` javascript
    // const tFromF = f => t // 注意tFromF在这里是 `纯函数` ，本身没有包装任何方法
    [
        periodFromOrdinal, // 注意去掉了一层调用，因为没参数。具体原因后面currying时解释
        voListFromPeriod,
        filterPeriodB,
        voFromVOList,
        doWithVOList,
        propFromVO,
        mapWithVO,
    ].reduce((last, chain) => chain(last), ordinal);
    // import * as R from 'ramda';
    require(['https://cdn.jsdelivr.net/npm/ramda@latest/dist/ramda.min.js'], R => window.R = R);
    // R.pipe(i => 1 + i, i => 3 * i)(1);
    // R.compose(i => 1 + i, i => 3 * i)(1);
    R.pipe(
        periodFromOrdinal,
        voListFromPeriod,
        filterPeriodB,
        voFromVOList,
        doWithVOList,
        propFromVO,
        mapWithVO,
    )(ordinal);
```

### 高阶函数与currying
函数：无状态
``` javascript
    function bad0() { return this; } // 这叫做方法，但是是unbind的 // [bad2][0]();
    const bad1 = ((i = 0) => () => i++)(); // 这叫做闭包 // bad3(); bad3();
```

纯函数：相同输入一定有相同输出
``` javascript
    // 无外部IO
    const bad0 = arr => arr[parseInt(arr.length * Math.random())]; // bad0([1, 2, 3]); bad0([1, 2, 3]);
    // 无副作用
    const bad1 = arr => arr.reverse(); // const arr = [1, 2, 3]; bad1(arr); arr; bad1(arr);
    // 当然，不可变
    let bad2 = () => { bad4 = () => 'not first'; return 'first'; }; // bad4();
```

高阶函数：生成函数的函数
``` javascript
    const greet = who => when => `good ${when}, ${who}`; // const greetAlice = greet('Alice'); greetAlice('morning'); greetAlice('afternoon');
```

匿名函数：
``` javascript
    var pow = (() => i => i ** 2)(); // p.name; (i => i ** 2).name;
```
``` elixir
    fn a, b -> a + b end
```

函数字面量：
``` elixir
    &(&1 + &2)
```

闭包：访问外部变量的函数
``` javascript
    const counter = ((i = 0) => () => i++)(); // counter(); counter(); counter();
```

Currying：偏函数
``` javascript
    // 用bind模拟 Iteratee-first, Data-last 的柯里化函数
    const curry = (a, ...args) => { const next = (b, ...args) => { const next = i => (i + a) * b; return args.length? next(...args) : next }; return args.length ? next(...args) : next; }; // curry(1)(2)(3); curry(1, 2, 3);
    // 试想如果对一个非纯函数进行Currying会发生什么？
```

### 链式、compose的语法层面支持：函数传递
``` elixir
    add = fn i, inc -> i + inc end
    multi = fn i, mul -> i * mul end
    pow = fn i -> i * i end
    # 1 |> add.(2) |> multi.(3) |> pow.()
```


***
## 逻辑同步的异步
Promise模型;
``` javascript
    const waitAMoment = new Promise(resolve => setTimeout(() => resolve('ready'), 3000));

    const then = waitAMoment.then(console.log);

    const thenDo = waitAMoment.then(() =>
        console.log('do something,')
    ).then(() =>
        new Promise(r => setTimeout(r, 3000))
    ).then(() =>
        console.log('then do something.')
    );

    const thenDoElse = waitAMoment.then(() =>
        console.log('something else,')
    ).then(() =>
        console.log('then something else.')
    );

    Promise.all([then, thenDo, thenDoElse]).then(() => console.log('fin.'));
```

`generator` 模型
``` python
def ping():
    input = yield
    print('[PING] u say: %s' % input)
    yield 'pong'

def pong(input):
    ge = ping()
    next(ge)
    print('[PONG] ping say: %s' % ge.send('ping'))
```

`async` 模型
``` javascript
    const ping = async input => (console.log(`[PING] u say: ${input}`), 'pong');
    const pong = async input => console.log(`[PONG] ping say: ${await ping(input)}`);
```

`async-io (async+corotine)`
``` javascript
    const waitAMoment = new Promise(resolve => setTimeout(() => resolve('ready'), 3000));
    
    const then = (async () => {
        const resp = await waitAMoment;
        console.log(resp);
    })();

    const thenDo = (async () => {
        await waitAMoment;
        console.log('do something,');
        await new Promise(r => setTimeout(r, 3000));
        console.log('then do something.');
    })();

    const thenDoElse = (async () => {
        await waitAMoment;
        console.log('something else,')
        console.log('then something else.')
    })();

    for (let task of [then, thenDo, thenDoElse]) {
        await task;
    }
    console.log('fin.');
```

`gorotine (corotine+channel)`: [playground](https://play.golang.org/p/VRP3922ulfI)
``` go
package main

import (
	"fmt"
)

func main() {
	in, out := make(chan string), make(chan string)
	ping := func() {
		input := <-in
		fmt.Println("[PING] u say:", input)
		out <- "pong"
	}
	pong := func(input string) {
		go ping()
		in <- "ping"
		output := <-out
		fmt.Println("[PONG] ping say:", output)
	}
	pong("ping")
	pong("ping")
	pong("ping")
}
```

尝试对以上例子中的 `ping` 与 `pong` 进行并发、并行改造。

`yield` 跳出到调用者（一种 `goto` ）， `await` 直接交出执行权， `go` 通过 `channel` 控制流程。个人以为后者更优。

`lightweight-process (corotine+process)`: [Processes](http://erlang.org/doc/efficiency_guide/processes.html)
> 注意尾递归问题！

`defer`
``` javascript
    // 从这个例子看为何async无法完全替代Promise。
    const { defers, returns } = await new Promise(factor => {
        const defers = new Promise(returns => factor({
            get defers() { return defers; },
            get returns() { return returns; },
        }));
    });
    defers.then(r => console.log('before return,', r));
    console.log('do something');
    returns('fin');
```
[playground](https://play.golang.org/p/Rle80dCXNKw)
``` go
	defer fmt.Println("before return.")
	fmt.Println("do something")
```


### 注意！
async架构的高度传染性：
> 如上面那些例子，一旦一个模块运用了 `async` 模型，其调用者也得变为 `async` ，层层向上，具有极强的侵入性。

通过Go看函数式：
> 不是以上所有东西都适合于所有场景，准确的说大部分场景其实用不上，过犹不及。`Go` 有闭包，有匿名函数高阶函数，但没有语言层面的Currying支持，保留了可变量、状态等，使用控制流程的if-else、for循环，等等。但仍然很适用于高并发场景。可见：1. 函数式一些概念确实很适合并发，2. 但并不是所有函数式概念都是并发必须的。


***
## 元编程
### 动态修改&访问拦截：[a == 1 && a == 2 && a == 3](https://zhuanlan.zhihu.com/p/33029291)
``` javascript
Object.defineProperty(this, 'a', { get: ((i = 0) => () => ++i)() });
```

### 扩展语言能力
``` javascript
((global, modules) =>
    Object.defineProperties(global, {
        exports: { set: mod => modules[mod.name] = mod },
        imports: { get: () => name => modules[name] },
    })
)(this, {});
```

### 面向程序编程
考虑之前 `compose` 的例子，下面我们体验一下用代码生成代码：
``` javascript
    const pipe = (...fns) => arg => fns.reduce((arg, fn) => fn(arg), arg);

    const bFromA = arg => `${arg}, A`;
    const cFromB = arg => `${arg}, B`;
    const dFromC = arg => `${arg}, C`;
    const eFromD = arg => `${arg}, D`;

    const filterA = arg => `${arg}, filterA`;
    const mapB = arg => `${arg}, mapB`;
    const doWithC = arg => `${arg}, doWithC`;

    const specials = [
        [bFromA, filterA, cFromB, dFromC, eFromD],
        [bFromA, cFromB, mapB, dFromC, eFromD],
        [bFromA, cFromB, dFromC, doWithC, eFromD],
        [bFromA, filterA, cFromB, dFromC, doWithC, eFromD],
    ].map(fns => fns.map(fn => fn.name));
    const funcs = { bFromA, cFromB, dFromC, eFromD, filterA, mapB, doWithC };

    const comb = entries => // 推导出所有组合
        entries.map((fn, index) => [[fn.name], ...comb([...entries.slice(0, index), ...entries.slice(1 + index, entries.length)])]);
    const flat = ([head, ...tail]) => // 每个大组的小组合扁平化
        tail.reduce(((r, [h, ...t]) =>
            [...r, ...(t.length ? flat([[...head, h[0]], ...t]) : [[...head, h[0]]])]
        ), []);

    pipe(
        comb,
        combs => combs.map(flat),
        combs => combs.reduce((r, t) => [...r, ...t], []), // 四个大组扁平化
        combs => [...combs, ...specials], // 加入特殊组合规则
        combs => combs.map(comb => [comb.join('_'), pipe(...comb.map(name => funcs[name]))]), // 推导出所有函数名与函数体
        fns => fns.reduce((dict, [name, fn]) => (dict[name] = fn, dict), {}), // 归并得到对象
    )([bFromA, cFromB, dFromC, eFromD]);
    // Promise.resolve([bFromA, cFromB, dFromC, eFromD]).then(comb).then(combs => combs.map(flat));
```

### 注意！代码展开的时机，只效率而言：编译时宏展开 > 运行前展开 > 初次执行延迟展开 > 每次执行展开

## 非面向对象的抽象
函数式没有面向对象，但不代表其抽象能力弱。

### 数据与方法的封装
传统面向对象：对象绑定方法
``` javascript
    const nodes = (div => (div.innerHTML = '<span>a</span><span>b</span><span>c</span>', div.querySelectorAll('span')))(document.createElement('div'));
    NodeList.prototype.some = Array.prototype.some; // 基于原型链的绑定
    nodes.some(console.log);
```

基于消息传递的面向对象：
> 其实按照面向对象之父（smalltalk，无条件，无循环）的说法，这才是真面向对象
``` javascript
    const some = Array.prototype.some;
    const send = (reciever, message, ...args) => message.call(reciever, ...args);
    send(nodes, some, console.log);
```
> 在支持消息的语言，这里的 `send` 方法是不必要的。可以参见上面那个 `lightweight-process` 的例子。

很明显，后者与函数式更亲和（你只需要声明函数，不需要声明每个对象具有什么函数，即chain与compose的区别）。

URL的查询（search）部分其实可以看作典型的消息传递， `schema://host:port/path/#hash?search` ，比如 `/sth?filter0=arg0&filter1=arg1` ，展开为类似的函数式代码：
``` elixir
query = sth |> filter0(arg0) |> filter1(arg1)
```

### 抽象与复用
传统面向对象：基于继承（ `extends` , `implements` ）的抽象
``` javascript
    class Paper {
        constructor(score: number) {
            this.score = score;
        }
        print() {
            console.log(this.score);
        }
        minced() { }
    }
    // class Student extends Paper { } // wtf, 人继承物？那么我可以调用 `student.minced();` 搅碎一个人咯？

    // class Score {}
    // class Paper extends Score {}
    // class Student extends Score {}
    // // 那么考虑多继承呢？
```
基于 `mixin` 或 `composition` 实现抽象：
``` javascript
    // 但 `mixin` 近来饱受质疑
    // function print() { console.log(this.score); } // 这不是纯函数，纯函数应该这么写： `const print = ({ score }) => console.log(score);`
    // const Paper = score => ({ score, print, minced: () => { } }); // 纯函数写法： `({ ..., print: () => print(this), ... })` ，但我是实用主义者。
    // const Student = score => ({ score, print });
    // const Cirno = score => ({ ...Student(Math.min(score, 9)) });

    const Score = score => ({ score, print() { console.log(this.score) } })
    const Paper = score => ({ ...Score(score), minced: () => { } });
    const Student = score => ({ ...Score(score) });
    const Cirno = score => ({ ...Student(Math.min(score, 9)) });

    Paper(80).print();
    Student(80).print();
    Cirno(80).print();
```
当然，这不代表只有函数式选择组合。 `Go` 对组合支持的非常彻底：[playground](https://play.golang.org/p/lNhMOASMB5-)
``` go
package main

import (
	"fmt"
	"math"
)

type Score float64

func (s Score) print() { fmt.Println(s) }

type Paper struct{ Score }
type Student struct{ Score }
type Cirno struct{ *Student }

func NewPaper(score float64) *Paper     { return &Paper{Score(score)} }
func NewStudent(score float64) *Student { return &Student{Score(score)} }
func NewCirno(score float64) *Cirno     { return &Cirno{NewStudent(math.Min(score, 9))} }

func main() {
	NewPaper(80).print()
	NewStudent(80).print()
	NewCirno(80).print()
}

```
继承：is，组合：in。


***
## 其它
### 逻辑编程
除了函数式外，最近另一个重新开始受到关注的编程泛型，很有助于理解模式匹配。
[playground](https://swish.swi-prolog.org/)
``` prolog
	uniq(_,[]).
	uniq(X,[H|T]) :- uniq(H, T), uniq(X, T), \+(X=H).
	uniq([H|T]) :- uniq(H,T).

    comb([bFromA]).
    comb([cFromB]).
    comb([dFromC]).
    comb([eFromD]).
    comb(L) :- L=[H|T], comb(T), comb([H]), uniq(L).

    | ?- comb([A, B, C, D]).
```

### 强弱类型
动态 != 弱类型，静态 != 强类型。尤其要注意，JS是弱类型，JSON也是（虽然严格来说JSON不算语言，只是JS的子集）。

JSON里没有int、long、float、double，一切都是number，所以就算你从Java往JSON里put一个long，拿出来也可能是int！
