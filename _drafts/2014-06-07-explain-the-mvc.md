# What is maintainable code?

http://singlepageappbook.com/:

- it is easy to understand and troubleshoot
- it is easy to test
- it is easy to refactor

# What is hard-to-maintain code?

- it has many dependencies, making it hard to understand and hard to test independently of the whole
- it accesses data from and writes data to the global scope, which makes it hard to consistently set up the same state for testing
- it has side-effects, which means that it cannot be instantiated easily/repeatably in a test
- it exposes a large external surface and doesn't hide its implementation details, which makes it hard to refactor without breaking many other components that depend on that public interface

# SOC

Separation of concerns

# OOD&OOP原则

S.O.L.I.D

- **S**: The Single Responsibility Principle 
    - 单一功能原则: 让一个类只做一种类型责任，当这个类需要承当其他类型的责任的时候，就需要分解这个类。

- **O**: The Open Closed Principle 
    - 开放封闭原则: 软件实体应该是可扩展，而不可修改的。也就是说，对扩展是开放的，而对修改是封闭的。

- **L**: The Liskov Substitution Principle
    - 里氏替换原则: 一个子类的实例应该能够替换任何其父类的实例


- **I**: The Dependency Inversion Principle
    - 依赖反转原则/依赖倒置/注入依赖: 1. 高层模块不应该依赖于低层模块，二者都应该依赖于抽象; 2. 抽象不应该依赖于细节，细节应该依赖于抽象


- **D**: The Interface Segregation Principle
    - 接口分离原则: 多个特定客户端接口要好于一个宽泛用途的接口

# Traditional MVC


# The most importent thing

- Responsibility
- Communication

