-  `zrender.Group.getBoundingRect(includeChildren)`

   得到 `includeChildren` 或组内所有元素的包围盒，但是这个包围盒不会随着scale的改变，而改变

-  关于`zrender.Transformable.origin` 的一些看法

   origin 存在时，position不会变

   origin 不存在时 会根据 rotation/scale，相对 origin 进行相对转换

   ​