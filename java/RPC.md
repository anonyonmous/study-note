# RPC
## 1.手写一个RPC Clinet
简单rpc框架demo https://github.com/yeecode/EasyRPC

```java
@RestController
public class MainController {
    @RequestMapping("/")
    public Result rpcMain(String identifier, String methodName, String argTypes, String argValues) {
        try {
            Class clazz = Class.forName(identifier);
            if (clazz == null) {
                return Result.getFailResult("目标类不存在");
            }

            List<String> argTypeList = JSON.parseArray(argTypes, String.class);
            List<Class> argClassList = new ArrayList<>();
            for (String argType : argTypeList) {
                argClassList.add(Class.forName(argType));
            }
            Class[] argClassArray = new Class[argClassList.size()];
            argClassList.toArray(argClassArray);

            List<String> argValueStringList = JSON.parseArray(argValues, String.class);
            List<Object> argValueList = new ArrayList<>();
            for (int i = 0; i < argTypeList.size(); i++) {
                if (argClassList.get(i).equals(String.class)) {
                    argValueList.add(argValueStringList.get(i));
                } else {
                    argValueList.add(JSON.parseObject(argValueStringList.get(i), argClassList.get(i)));
                }
            }
            Object[] args = new Object[argValueList.size()];
            argValueList.toArray(args);

            Method method = clazz.getMethod(methodName, argClassArray);
            if (method == null) {
                return Result.getFailResult("目标方法不存在");
            }

            Object obj = ServiceGetter.getServiceByClazz(clazz);
            if (obj == null) {
                return Result.getFailResult("目标类的实例无法生成");
            }
            Object result = method.invoke(obj, args);
            return Result.getSuccessResult(method.getReturnType().getName(), JSON.toJSONString(result));
        } catch (Exception ex) {
            ex.printStackTrace();
            return Result.getFailResult("服务端解析异常");
        }
    }
}
```

## 2.RPC协议



## 3.RPC和SOA的区别

> RPC(Remote Procedure Call Protocol) 远程过程调用协议，一个应用部署在A上想要调用B服务器应用提供的函数方法，
通过网络来表达调用的语义和传达调用的数据。

> SOA（Service Oriented Ambiguity） 即面向服务架构 是一种设计方法，其中包含多个服务，而服务之间通过配合最终会提供一系列功能。一个服务通常以独立的形式存在于操作系统进程中。服务之间通过网络调用，而非采用进程内调用的方式进行通信。

RPC
![StudyNote](image/WeChat9bc2dbb3c7d285181461243ea40d486d.png)

SOA
![StudyNote](image/WeChat779515736597abb4b600008e9086ee4a.png)


## 4.既有 HTTP ,为啥用 RPC 进行服务调用?

RPC 只是一种设计而已RPC 只是一种概念、一种设计，就是为了解决 不同服务之间的调用问题, 它一般会包含有 传输协议 和 序列化协议 这两个。但是，HTTP 是一种协议，RPC框架可以使用 HTTP协议作为传输协议或者直接使用TCP作为传输协议，使用不同的协议一般也是为了适应不同的场景。