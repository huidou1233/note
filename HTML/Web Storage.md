作为 Web Storage API 的接口，`Storage` 提供了访问特定域名下的会话存储（session storage）或本地存储（local storage）的功能，例如，可以添加、修改或删除存储的数据项。

如果你想要操作一个域名的会话存储（session storage），可以使用Window.sessionStorage；如果想要操作一个域名的本地存储（local storage），可以使用Window.localStorage。

属性：

​	Storage.length: 表示目前Storage对象中存储到的键-值对数量。Storage对象是同源的，这意味着Storage对象的项数(和长度)只反映同源情况下的项数。

方法：

​	Storage.key(index): 允许获取一个指定位置的键。

​	Storage.getItem(key): 根据给定的键返回相应的值。

​	Storage.setItem(key,value): 将数据存入指定键对应的位置，如果键名存在，则更新其对应的值。值得注意的是，如果用户关闭了网站的存储，或存储已达到其最大容量，那么此时设置将抛出错误，因此，在需要设置数据的场合，务必保证应用程序能够处理此类异常。

​	Storage.removeItem(key): 删除key对应的数据项，如果key值不存在，则不执行任何操作

​	Storage.clear(): 删除存储列表中的所有数据。

sessionStorage和localStorage

| sessionStorage                           | localStorage       |
| ---------------------------------------- | ------------------ |
| 数据会保存到存储他的窗口或标签页关闭时(浏览器刷新时可以存储数据，关闭是不可以) | 数据的生命期比窗口或浏览器的生命期长 |
| 数据只在构建它们的窗口或者标签页内可见                      | 数据可被同源的每个窗口或者标签页共享 |

浏览器有时会重新定义窗口或者标签的生命周期。例如，浏览器崩溃或者用户关闭已打开的多个标签页是，一些浏览器会保存并恢复当前会话。在这些情况下，浏览器重启或恢复时，可能会选择保存相关的sessionStorage