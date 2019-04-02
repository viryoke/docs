1、AIDL：Android Interface Definition Language（Android接口定义语言），用于进程间通信；
2、暴露给其他应用进行调用的应用称为服务端，调用其他应用的方法的应用称为客户端，客户端通过绑定服务端的Service来进行交互；
3、AIDL接口参数支持List（支持泛型）和Map（不支持泛型），服务端必须使用ArrayList和HashMap接收；
4、定向Tag，分为：in out inout三种；
