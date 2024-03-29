### 虚拟栈

lua调用C的函数都会得到一个新的虚拟栈，独立于之前的栈

C调用lua，每一个协程都有一个栈(需要注意爆栈的问题)

### Lua调C

c编写源文件并编译成动态库，lua从动态库中寻找luaopen_*命名的函数

```
local tbl=require "tabl.c"//去动态库加载luaopen_tabl_c函数
static const luaL_Reg l[]={{"echo",lecho}}
```

### 闭包

```
lua_pushcclosure用来创建C闭包
lua_upvalueindex伪索引来获取上值
可以为多个导出函数（c导出函数给lua使用）共享上值，可以减少传递一个参数
```

```
#include <lua.h>
#include <lauxlib.h>
#include <lualib.h>
#include <stdio.h>
// 闭包实现：  函数 + 上值  luaL_setfuncs
// lua_upvalueindex(1)
// lua_upvalueindex(2)
static int
lecho (lua_State *L) {
	lua_Integer n = lua_tointeger(L, lua_upvalueindex(1));//把上值取出
    n++;
    const char* str = lua_tostring(L, -1);
    fprintf(stdout, "[n=%lld]---%s\n", n, str);
    lua_pushinteger(L, n);
    lua_replace(L, lua_upvalueindex(1));
    return 0;
}

static const luaL_Reg l[] = {
	{"echo", lecho},
	{NULL, NULL},
};

int
luaopen_uv_c(lua_State *L) { // local tbl = require "tbl.c"
	luaL_newlibtable(L, l);// 1
    lua_pushinteger(L, 0);// 2
	luaL_setfuncs(L, l, 1);// 上值
    // luaL_newlib(L, l);
	return 1;
}

```

### 注册表

可以用来在多个C库中共享lua数据(包括userdata和lightuerdata)

- 一张预定义的表
- 使用LUA_REGISTRYINDEX来索引