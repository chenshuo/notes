# Notes on Lua implementation

Releases:

![releases](lua/releases.png)

[Calltree of Lua 5.3](lua/calltree.html) for following simple program:

```c++
#include "lua.hpp"

int main()
{
  lua_State *L = luaL_newstate();
  luaL_openlibs(L);  // open standard libraries
  luaL_loadstring(L, R"(
      x = 10 * 5
      print(x)
      )");
  lua_pcall(L, 0, LUA_MULTRET, 0);
  lua_close(L);
}
```
