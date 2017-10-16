Element和Instance的区别有什么？
没有继承React的class是什么意思？
https://stackoverflow.com/questions/43703984/react-component-without-extend-react-component-class
https://stackoverflow.com/questions/36296658/do-not-extend-react-component

this.setState是异步的
当props发生更改时，componentWillReceiveProps会被调用；但是并不意味着componentWillReceiveProps被调用了而props发生了更改。也就是在一些情况下，componentWillReceiveProps被调用了，但是props并没有发生更改
https://reactjs.org/blog/2016/01/08/A-implies-B-does-not-imply-B-implies-A.html
React.PureComponent

render返回的是什么？是element吧？
render pass？

需要实验的部分：key以及内存回收