

## yaml语法

 * 大小写敏感

 * 使用缩进表示层级关系

 * 缩进不允许使用tab，只允许空格

 * 缩进的空格数不重要，只要相同层级的元素左对齐即可

 * #表示注释

 * 键和值用：隔开且：后必须跟上一个空格

 * 字符串类型不一定使用双引号标识

 * YAML也有用来描述好几行相同结构的数据的缩写语法，数组用'[]'包括起来，hash用'{}'来包括。因此，上面的这个YAML能够缩写成这样:

   ```yaml
   house:
     family: { name: Doe, parents: [John, Jane], children: [Paul, Mark, Simone] }
     address: { number: 34, street: Main Street, city: Nowheretown, zipcode: 12345 }
   ```

