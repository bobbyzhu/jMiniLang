
# jMiniLang - A Simple Interpreter (*Java*)
===========================

***jMiniLang*** is a simplified interpreter framework. Developed by ***bajdcc***.
*PS.* ***LR Analysis*** refers to ***VFS*** developed by [*vczh*](https://github.com/vczh "Github page of vczh").

#### Features

1. Lexer which generates *NFA* and *DFA*.
2. Parser which implements *LALR(1) Grammar* with optional semantic action.
3. Customized semantic analysis.
4. Printable syntax tree.
5. Stack-based instruction design.
6. Native method.
7. Import and export of code page.
8. Serializable code page.
9. **Lambda functions and Closure**.
10. Display grammar and semantic errors.
11. **Pipe**.
12. **Multiple process**.
13. Load file with Sync/Async.
14. **Virtual machine**.
15. **Support GUI**.
16. **Functional programming**.

#### What it generates

- Structures of *Regex*, *NFA* and *DFA* Table.
- Structures of *LL*/*LR* Table.
- Structures of semantic instructions.
- Structures of syntax tree.
- Structures of code page.
- Virtual machine instructions.
- Runtime environment.

#### Virtual Machine OS

An OS running on *jMiniLang* interpreter.

Now has commands:
- echo
- dup
- pipe
- grep
- proc
- range
- task
- sleep
- time
- count

Tasks:
- System
- Utility
- Remote
- UI

UI:
- Clock
- Hitokoto

Implemented IPC, usage:
- `task system now` -> Get system time
- `task util calc 1+2*3` -> Val = 7
- `task ui print hello world` -> Remote window
- `task ui path M 100 100 L 200 200` -> SVG

Utility:
- `task util doc g_func_fold` -> Document
- `task util reverse ...`
- `task util toupper ...`
- `task util sum ...`
- `task util product ...`
- `task util palindrome ...`

#### Manual

[*Simplified Chinese Version*](https://raw.githubusercontent.com/bajdcc/jMiniLang/master/%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.pdf "Manual - Simplified Chinese")

#### Example

**1. Lambda: Y Combinator of Hanoi**

*Code:*

```javascript
import "sys.base";
var move = func ~(i, x, y) {
    call g_printn(call g_to_string(i) + ": " + 
        call g_to_string(x) + " -> " + call g_to_string(y));
};
var hanoi = func ~(f) {
    var fk = func ~(i, a, b, c) {
        if (i == 1) {
            call move(i, a, c);
        } else {
            call f(i - 1, a, c, b);
            call move(i, a, c);
            call f(i - 1, b, a, c);
        }
    };
    return fk;
};
var h = call (func ~(f) {
    var fx = func ~(x) {
        var fn = func ~(i, a, b, c) {
            var vf = call f(call x(x));
            return call vf(i, a, b, c);
        };
        return fn;
    };
    return call (func ~(h) -> call h(h))(fx);
})(hanoi);
call h(3, 'A', 'B', 'C');
```

*Result:*

```c
1: A -> C
2: A -> B
1: C -> B
3: A -> C
1: B -> A
2: B -> C
1: A -> C
```

**2. Lambda: Trampoline**

*Code:*

```javascript
import "sys.base";
call g_printn("Trampoline example:");
var repeat = func ~(operation, count) {
    var repeat0 = func ~() {
        if (count <= 0) { return; }
        call operation(count);
        return call repeat(operation, --count);
    };
    return repeat0;
};
var print = func ~(n) -> call g_printn("n: " + n);
var tfun = func ~() -> call repeat(print, 5);
call(func ~(f) {
    while (!(call g_is_null(f)) && (call g_get_type(f) == "函数")) {
        let f = call f(); // Trampoline, like CPS.
    }
})(tfun);
```

*Result:*

```c
Trampoline example:
n: 5
n: 4
n: 3
n: 2
n: 1
```

**3. List: LinkedList**

*Code:*

```javascript
import "sys.base";
import "sys.list";
var create_node = func ~(data) {
    var new_node = g_new_map;
    call g_map_put(new_node, "data", data);
    call g_map_put(new_node, "prev", g_null);
    call g_map_put(new_node, "next", g_null);
    return new_node;
};
var append = func ~(head, obj) {
    var new_node = call create_node(obj);
    call g_map_put(new_node, "next", head);
    call g_map_put(head, "prev", new_node);
    return new_node;
};
var head = call create_node(0);
foreach (var i : call g_range(1, 10)) {
    let head = call append(head, i);
}
var p = head;
while (!call g_is_null(p)) {
    call g_printn(call g_map_get(p, "data"));
    let p = call g_map_get(p, "next");
}
```

*Result:*

```c
10
9
8
7
6
5
4
3
2
1
0
```

**4. Multi-Process: Pipe**

*Code:*

```javascript
import "sys.base";
import "sys.proc";
var proc = func ~() {
    var handle = call g_create_pipe("test");
    var print = func ~(ch) -> call g_print(ch);
    var pid = call g_get_pid();
    if (pid == 0) {
        foreach (var i : call g_range(1, 10)) {
            call g_printn("[" + call g_get_pid() + "] Hello world!");
            call g_write_pipe(handle, "" + i);
        }
        call g_destroy_pipe(handle);
    } else {
        call g_printn("[" + call g_get_pid() + "] Hello world!");
        call g_read_pipe(handle, print);
    }
};
call g_create_process(proc);
call proc();
```

*Result:*

```c
[1] Hello world!
[0] Hello world!
1[0] Hello world!
2[0] Hello world!
3[0] Hello world!
[0] Hello world!
4[0] Hello world!
5[0] Hello world!
6[0] Hello world!
7[0] Hello world!
8[0] Hello world!
91
```

**5. Multi-Process: Consumer-Producer Model**

*Code:*

```javascript
import "sys.base";
import "sys.list";
import "sys.proc";

var goods = [];
call g_start_share("goods", goods);
var index = 1;
call g_start_share("index", index);
var consumer = func ~() {
    for (;;) {
        var goods = call g_query_share("goods");
        if (call g_is_null(goods)) {
            break;
        }
        var g = call g_array_pop(goods);
        if (!call g_is_null(g)) {
            call g_printn("Consumer#" + call g_get_pid() + " ---- get " + g);
        }
    }
    call g_printn("Consumer#" + call g_get_pid() + " exit");
};
var producer = func ~() {
    foreach (var i : call g_range(1, 5)) {
        var goods = call g_reference_share("goods");
        call g_lock_share("index");
        var index = call g_reference_share("index");
        call g_printn("Producer#" + call g_get_pid() + " ++++ put " + index);
        call g_array_add(goods, index);
        index++;
        call g_stop_share("index");
        call g_unlock_share("index");
        call g_stop_share("goods");
    }
    call g_printn("Producer#" + call g_get_pid() + " exit");
};
var create_consumer = func ~(n) {
    var handles = [];
    foreach (var i : call g_range(1, n)) {
        var h = call g_create_process(consumer);
        call g_array_add(handles, h);
        call g_printn("[" + i + "] Create consumer: #" + h);
    }
    return handles;
};
var create_producer = func ~(n) {
    var handles = [];
    foreach (var i : call g_range(1, n)) {
        var h = call g_create_process(producer);
        call g_array_add(handles, h);
        call g_printn("[" + i + "] Create producer: #" + h);
    }
    return handles;
};

var consumers = call create_consumer(3);
var producers = call create_producer(4);
call g_printn("Waiting for producers to exit...");
call g_join_process_array(producers);
call g_printn("Producers exit");
call g_printn("Waiting for consumers to exit...");
call g_stop_share("index");
call g_stop_share("goods");
call g_join_process_array(consumers);
call g_printn("Consumers exit");
```

*Result:*

```c
[1] Create consumer: #1
[2] Create consumer: #2
[3] Create consumer: #3
[1] Create producer: #4
[2] Create producer: #5
Producer#4 ++++ put 1
Consumer#3 ---- get 1
[3] Create producer: #6
Producer#5 ++++ put 2
[4] Create producer: #7
Consumer#2 ---- get 2
Producer#4 ++++ put 3
Waiting for producers to exit...
Consumer#1 ---- get 3
Producer#7 ++++ put 4
Consumer#2 ---- get 4
Producer#7 ++++ put 5
Consumer#3 ---- get 5
Producer#5 ++++ put 6
Consumer#2 ---- get 6
Producer#5 ++++ put 7
Consumer#1 ---- get 7
Producer#7 ++++ put 8
Consumer#3 ---- get 8
Producer#6 ++++ put 9
Consumer#3 ---- get 9
Producer#5 ++++ put 10
Consumer#2 ---- get 10
Producer#7 ++++ put 11
Consumer#1 ---- get 11
Producer#4 ++++ put 12
Consumer#3 ---- get 12
Producer#5 ++++ put 13
Consumer#1 ---- get 13
Producer#5 exit
Producer#6 ++++ put 14
Consumer#2 ---- get 14
Producer#4 ++++ put 15
Consumer#3 ---- get 15
Producer#7 ++++ put 16
Consumer#2 ---- get 16
Producer#7 exit
Producer#6 ++++ put 17
Consumer#1 ---- get 17
Producer#4 ++++ put 18
Consumer#1 ---- get 18
Producer#4 exit
Producer#6 ++++ put 19
Consumer#1 ---- get 19
Producer#6 ++++ put 20
Consumer#3 ---- get 20
Producer#6 exit
Producers exit
Waiting for consumers to exit...
Consumer#3 exit
Consumer#1 exit
Consumer#2 exit
Consumers exit
```

**6. Multi-Process: PC and Router**

*Code:*

```javascript
import "sys.base";
import "sys.list";
import "sys.proc";

var pc_router = 5;
call g_start_share("pc_router", pc_router);

var pc = func ~(index) {
    var pc_router = call g_query_share("pc_router");
    var name = "pc_" + index;
    var router = index / pc_router;
    var display = "PC #" + index;
    call g_printn(display + " started");
    call g_sleep(50);
    var handle = call g_create_pipe(name);
    call g_printn(display + " connecting...");
    var router_connection = "router#" + router;
    for (;;) {
        call g_sleep(100);
        call g_lock_share(router_connection);
        var connection = call g_query_share(router_connection);
        if (call g_is_null(connection)) {
            call g_unlock_share(router_connection);
            continue;
        }
        call g_printn(display + " connecting to #" + router);
        call g_array_add(connection, index);
        call g_unlock_share(router_connection);
        break;
    }
    var get_id = func ~(ch) {
        if (ch == '@') {
            call g_printn(display + " connected to router #" + router);
        }
    };
    call g_read_pipe(handle, get_id);
    call g_sleep(1000);
    call g_printn(display + " stopped");
};

var router = func ~(index) {
    var pc_router = call g_query_share("pc_router");
    var name = "router_" + index;
    var display = "Router #" + index;
    call g_printn(display + " started");
    var router_connection = "router#" + index;
    var connection = [];
    var list = [];
    call g_start_share(router_connection, connection);
    var connected = 0;
    var handle_pc = func ~(args) {
        var connected = call g_array_get(args, 0);
        var pc_router = call g_array_get(args, 1);
        var router_connection = call g_array_get(args, 2);
        var display = call g_array_get(args, 3);
        var list = call g_array_get(args, 4);
        for (;;) {
            call g_sleep(100);
            call g_lock_share(router_connection);
            var connection = call g_query_share(router_connection);
            var new_pc = call g_array_pop(connection);
            if (call g_is_null(new_pc)) {
                call g_unlock_share(router_connection);
                continue;
            }
            connected++;
            call g_array_add(list, new_pc);
            call g_printn(display + " connecting to pc #" + new_pc);
            call g_unlock_share(router_connection);
            var name = "pc_" + new_pc;
            var handle = call g_create_pipe(name);
            call g_destroy_pipe(handle);
            call g_printn(display + " connected to #" + new_pc);
            if (connected == pc_router) { break; }
        }
    };
    var args = [];
    call g_array_add(args, connected);
    call g_array_add(args, pc_router);
    call g_array_add(args, router_connection);
    call g_array_add(args, display);
    call g_array_add(args, list);
    call g_join_process(call g_create_process_args(handle_pc, args));
    var stop_pc = func ~(args) {
        var display = call g_array_get(args, 3);
        var list = call g_array_get(args, 4);
        var size = call g_array_size(list);
        for (var i = 0; i < size; i++) {
            call g_sleep(10);
            var name = "pc_" + call g_array_get(list, i);
            var handle = call g_create_pipe(name);
            call g_write_pipe(handle, "!");
            call g_printn(display + " disconnected with #" + i);
        }
    };
    call g_sleep(100);
    call g_join_process(call g_create_process_args(stop_pc, args));
    call g_printn(display + " stopped");
};

var create_pc = func ~(n) {
    var handles = [];
    foreach (var i : call g_range(0, n - 1)) {
        var h = call g_create_process_args(pc, i);
        call g_array_add(handles, h);
        call g_printn("Create pc: #" + i);
    }
    return handles;
};
var create_router = func ~(n) {
    var handles = [];
    foreach (var i : call g_range(0, n - 1)) {
        var h = call g_create_process_args(router, i);
        call g_array_add(handles, h);
        call g_printn("Create router: #" + i);
    }
    return handles;
};

call g_printn("Starting pc...");
var pcs = call create_pc(5);
call g_printn("Starting router...");
var routers = call create_router(1);
call g_join_process_array(pcs);
call g_join_process_array(routers);
```

*Result:*

```c
Starting pc...
Create pc: #0
PC #0 started
Create pc: #1
PC #1 started
Create pc: #2
PC #2 started
Create pc: #3
PC #3 started
Create pc: #4
PC #4 started
Starting router...
Create router: #0
Router #0 started
PC #0 connecting...
PC #1 connecting...
PC #2 connecting...
PC #3 connecting...
PC #4 connecting...
PC #0 connecting to #0
PC #1 connecting to #0
PC #2 connecting to #0
PC #3 connecting to #0
PC #4 connecting to #0
Router #0 connecting to pc #4
Router #0 connected to #4
Router #0 connecting to pc #3
Router #0 connected to #3
Router #0 connecting to pc #2
Router #0 connected to #2
Router #0 connecting to pc #1
Router #0 connected to #1
Router #0 connecting to pc #0
Router #0 connected to #0
Router #0 disconnected with #0
Router #0 disconnected with #1
Router #0 disconnected with #2
Router #0 disconnected with #3
Router #0 disconnected with #4
Router #0 stopped
PC #4 stopped
PC #3 stopped
PC #2 stopped
PC #1 stopped
PC #0 stopped
```

**7. Functional programming**

```
import "sys.base";
import "sys.list";
var g_func_max = func ~(a, b) -> a > b ? a : b;
var g_func_min = func ~(a, b) -> a < b ? a : b;
var g_func_lt = func ~(a, b) -> a < b;
var g_func_lte = func ~(a, b) -> a <= b;
var g_func_gt = func ~(a, b) -> a > b;
var g_func_gte = func ~(a, b) -> a >= b;
var g_func_eq = func ~(a, b) -> a == b;
var g_func_neq = func ~(a, b) -> a != b;
var g_func_add = func ~(a, b) -> a + b;
var g_func_sub = func ~(a, b) -> a - b;
var g_func_mul = func ~(a, b) -> a * b;
var g_func_div = func ~(a, b) -> a / b;
var g_func_and = func ~(a, b) -> a && b;
var g_func_or = func ~(a, b) -> a || b;
export "g_func_max";
export "g_func_min";
export "g_func_lt";
export "g_func_lte";
export "g_func_gt";
export "g_func_gte";
export "g_func_eq";
export "g_func_neq";
export "g_func_add";
export "g_func_sub";
export "g_func_mul";
export "g_func_div";
export "g_func_and";
export "g_func_or";
var g_func_curry = func ~(a, b) {
    var _curry = func ~(c) -> call a(b, c);
    return _curry;
};
var g_func_swap = func ~(a) {
    var _swap = func ~(b, c) -> call a(c, b);
    return _swap;
};
export "g_func_curry";
export "g_func_swap";
var g_func_1 = func ~(a) -> a;
export "g_func_1";
var g_func_always_1 = func ~(a) -> 1;
export "g_func_always_1";
var g_func_always_true = func ~(a) -> true;
export "g_func_always_true";

var g_func_xsl = func ["数组遍历闭包-foldl"] ~(l) {
    var len = call g_array_size(l);
    var idx = 0;
    var _xsl = func ~() {
        if (idx == len) { return g__; }
        var d = call g_array_get(l, idx);
        idx++;
        var _xsl_ = func ~() -> d;
        return _xsl_;
    };
    return _xsl;
};
export "g_func_xsl";
var g_func_xsr = func ["数组遍历闭包-foldr"] ~(l) {
    var idx = call g_array_size(l) - 1;
    var _xsr = func ~() {
        if (idx < 0) { return g__; }
        var d = call g_array_get(l, idx);
        idx--;
        var _xsr_ = func ~() -> d;
        return _xsr_;
    };
    return _xsr;
};
export "g_func_xsr";
// ----------------------------------------------
var g_func_fold = func 
    [
        "函数名：g_func_fold",
        "参数解释：",
        "  - name: 套用的折叠函数",
        "  - list: 需处理的数组",
        "  - init: 初始值(不用则为空)",
        "  - xs: 数组遍历方式(xsl=从左到右,xsr=从右到左)",
        "  - map: 对遍历的每个元素施加的变换",
        "  - arg: 对二元操作进行包装(默认=g_func_1,例=g_func_swap)",
        "  - filter: 对map后的元素进行过滤(true则处理)"
    ]
    ~(name, list, init, xs, map, arg, filter) {
    var len = call g_array_size(list);
    if (len == 0) { return g__; }
    var val = g__;
    var x = g__;
    if (call g_is_null(init)) {
        if (len == 1) { return call g_array_get(list, 0); }
        let x = call xs(list);
        let val = call x();
        let val = call val();
        let val = call map(val);
    } else {
        let x = call xs(list);
        let val = init;
    }
    var n = name;
    let n = call arg(n);
    for (;;) {
        var v2 = call x();
        if (call g_is_null(v2)) { break; }
        let v2 = call v2();
        let v2 = call map(v2);
        if (call filter(v2)) {
            let val = call n(val, v2);
        }
    }
    return val;
};
export "g_func_fold";
// ----------------------------------------------
var g_func_apply = func ~(name, list) ->
    call g_func_apply_arg(name, list, "g_func_1");
export "g_func_apply";
var g_func_apply_arg = func ~(name, list, arg) ->
    call g_func_fold(name, list, g__, "g_func_xsl", "g_func_1", arg, "g_func_always_true");
export "g_func_apply_arg";
var g_func_applyr = func ~(name, list) ->
    call g_func_applyr_arg(name, list, "g_func_1");
export "g_func_applyr";
var g_func_applyr_arg = func ~(name, list, arg) ->
    call g_func_fold(name, list, g__, "g_func_xsr", "g_func_1", arg, "g_func_always_true");
export "g_func_applyr_arg";
// ----------------------------------------------
var g_func_map = func ~(list, arg) ->
    call g_func_fold("g_array_add", list, g_new_array, "g_func_xsl", arg, "g_func_1", "g_func_always_true");
export "g_func_map";
var g_func_mapr = func ~(list, arg) ->
    call g_func_fold("g_array_add", list, g_new_array, "g_func_xsr", arg, "g_func_1", "g_func_always_true");
export "g_func_mapr";
var g_func_length = func ~(list) ->
    call g_func_fold("g_func_add", list, 0, "g_func_xsl", "g_func_always_1", "g_func_1", "g_func_always_true");
export "g_func_length";
var g_func_filter = func ~(list, filter) ->
    call g_func_fold("g_array_add", list, g_new_array, "g_func_xsl", "g_func_1", "g_func_1", filter);
export "g_func_filter";
// ----------------------------------------------
var take_filter = func ~(n) {
    var idx = 0;
    var end = n;
    var _take_filter = func ~(a) -> idx++ <= end;
    return _take_filter;
};
var drop_filter = func ~(n) {
    var idx = 0;
    var end = n;
    var _drop_filter = func ~(a) -> idx++ > end;
    return _drop_filter;
};
var g_func_take = func ~(list, n) ->
    call g_func_fold("g_array_add", list, g_new_array, "g_func_xsl", "g_func_1", "g_func_1", call take_filter(n));
export "g_func_take";
var g_func_taker = func ~(list, n) ->
    call g_func_fold("g_array_add", list, g_new_array, "g_func_xsr", "g_func_1", "g_func_1", call take_filter(n));
export "g_func_taker";
var g_func_drop = func ~(list, n) ->
    call g_func_fold("g_array_add", list, g_new_array, "g_func_xsl", "g_func_1", "g_func_1", call drop_filter(n));
export "g_func_drop";
var g_func_dropr = func ~(list, n) ->
    call g_func_fold("g_array_add", list, g_new_array, "g_func_xsr", "g_func_1", "g_func_1", call drop_filter(n));
export "g_func_dropr";
// ----------------------------------------------
var func_zip = func ~(name, a, b, xs) {
    var val = [];
    var xa = call xs(a);
    var xb = call xs(b);
    for (;;) {
        var _a = call xa();
        var _b = call xb();
        if (call g_is_null(_a) || call g_is_null(b)) {
            break;
        }
        var c = call name(call _a(), call _b());
        call g_array_add(val, c);
    }
    return val;
};
var g_func_zip = func ~(name, a, b) ->
    call func_zip(name, a, b, "g_func_xsl");
export "g_func_zip";
var g_func_zipr = func ~(name, a, b) ->
    call func_zip(name, a, b, "g_func_xsr");
export "g_func_zipr";
// ----------------------------------------------
var g_func_applicative = func ~(f, a, b) -> call f(a, call b(a));
export "g_func_applicative";
var g_func_import_string_module = func ~() { import "sys.string"; };
export "g_func_import_string_module";
```

#### Screenshot

*Screenshot 1 - Code*
![Screenshot 1](https://bajdcc.github.io/host/screenshot/jMiniLang_1.png)

*Screenshot 2 - Results*
![Screenshot 2](https://bajdcc.github.io/host/screenshot/jMiniLang_2.png)

*Screenshot 3 - Y-Combinator*
![Screenshot 3](https://bajdcc.github.io/host/screenshot/jMiniLang_3.png)

*Screenshot 4 - OS Virtual Machine with GUI*
![Screenshot 4](https://bajdcc.github.io/host/screenshot/jMiniLang_4.png)

*Screenshot 5 - Remote window*
![Screenshot 5](https://pic1.zhimg.com/v2-5072f9061e3bd1c5d8457bdeff24064c_r.png)

*Screenshot 6 - Functional programming*
![Screenshot 6](https://pic4.zhimg.com/v2-3b2c7ba9dd8d494555bd2b260c07e87f_r.png)
