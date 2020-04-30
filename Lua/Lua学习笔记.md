# Lua笔记

## 面向对象

```
myshape=Shape:new(nil,5)
myshape:printArena()
Square=Shape:new()
function Square:new( o,side )
  o=o or Shape:new(o,side)
  setmetatable(o)
end

```