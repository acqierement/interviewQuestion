# dubbo服务暴露

**ServiceConfig.doExportUrls()**

```java
private void doExportUrls() {
    List<URL> registryURLs = loadRegistries(true);
    for (ProtocolConfig protocolConfig : protocols) {
        String pathKey = URL.buildKey(getContextPath(protocolConfig).map(p -> p + "/" + path).orElse(path), group, version);
        ProviderModel providerModel = new ProviderModel(pathKey, ref, interfaceClass);
        ApplicationModel.initProviderModel(pathKey, providerModel);
        doExportUrlsFor1Protocol(protocolConfig, registryURLs);
    }
}
```

可以看到 Dubbo 支持多注册中心，并且支持多个协议，一个服务如果有多个协议那么就都需要暴露，比如同时支持 dubbo 协议和 hessian 协议，那么需要将这个服务用两种协议分别向多个注册中心（如果有多个的话）暴露注册。

**ServiceConfig#doExportUrlsFor1Protocol**

```java
        String name = protocolConfig.getName();
        if (StringUtils.isEmpty(name)) {
            name = DUBBO;
        }

        Map<String, String> map = new HashMap<String, String>();
        map.put(SIDE_KEY, PROVIDER_SIDE);
	// 这里主要是给map塞值进去。
        appendRuntimeParameters(map);
        appendParameters(map, metrics);
		......
		// 主要暴露流程在这里
        // export service
        String host = this.findConfigedHosts(protocolConfig, registryURLs, map);
        Integer port = this.findConfigedPorts(protocolConfig, name, map);
        URL url = new URL(name, host, port, getContextPath(protocolConfig).map(p -> p + "/" + path).orElse(path), map);

        Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(EXPORT_KEY,url.toFullString()));
        DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

        Exporter<?> exporter = protocol.export(wrapperInvoker);
        exporters.add(exporter);
        this.urls.add(url);
    }
```

这里我省略了超多代码，主要逻辑就是先生成url，然后将ref封装成Invoker。再把invoker包装成wrapperInvoker。之后转成export。

这里生成的url是dubbo开头的dubbo协议，而后面封装成invoker的时候会调用registryURL.addParameterAndEncoded(EXPORT_KEY,url.toFullString()，变成这样子的：

```
registry://127.0.0.1:2181/com.alibaba.dubbo.registry.RegistryService?application=demo-provider&dubbo=2.0.2&export=dubbo://192.168.1.17:20880/com.alibaba.dubbo.demo.DemoService....
```

可以看到走 registry 协议，然后参数里又有 export=dubbo://，这个走 dubbo 协议。

所以我们可以得知会先通过 registry 协议找到  RegistryProtocol 进行 export，并且在此方法里面还会根据 export 字段得到值然后执行 DubboProtocol 的 export 方法。

先看看RegistryProtocol的export方法，注册就是在这里实现的。

**RegistryProtocol#export()**

```java
    @Override
    public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException {
        URL registryUrl = getRegistryUrl(originInvoker);
        // url to export locally
        URL providerUrl = getProviderUrl(originInvoker);
		......

        //暴露invoker 
        final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker, providerUrl);

        final Registry registry = getRegistry(originInvoker);
        final URL registeredProviderUrl = getUrlToRegistry(providerUrl, registryUrl);
        // decide if we need to delay publish
        boolean register = providerUrl.getParameter(REGISTER_KEY, true);
        if (register) {
            // 这里执行注册url操作
            register(registryUrl, registeredProviderUrl);
        }
		......
        notifyExport(exporter);
        //Ensure that a new exporter instance is returned every time export
        return new DestroyableExporter<>(exporter);
    }
```

doLocalExport主要是将上面的 export=dubbo://... 先转换成 exporter。里面会去调用 DubboProtocol#export 方法。

然后会通过register方法将服务注册。

**RegistryProtocol#register**

```java
private void register(URL registryUrl, URL registeredProviderUrl) {
    Registry registry = registryFactory.getRegistry(registryUrl);
    registry.register(registeredProviderUrl);
}
```

如果是zk的就会调用zk的register方法，生成一个文件目录。

**ZookeeperRegistry#doRegister**

```java
@Override
public void doRegister(URL url) {
    try {
        zkClient.create(toUrlPath(url), url.getParameter(DYNAMIC_KEY, true));
    } catch (Throwable e) {
        throw new RpcException("Failed to register " + url + " to zookeeper " + getUrl() + ", cause: " + e.getMessage(), e);
    }
}
```

前面提到了`RegistryProtocol#export()`方法里会调用DubboProtocol的export方法

**DubboProtocol#export**

```java
@Override
public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
    URL url = invoker.getUrl();

    // export service.
    String key = serviceKey(url);
    DubboExporter<T> exporter = new DubboExporter<T>(invoker, key, exporterMap);
    exporterMap.put(key, exporter);

    //export an stub service for dispatching event
	......
	
    // 打开server
    openServer(url);
    optimizeSerialization(url);

    return exporter;
}
```

 export 主要就是根据 URL 构建出 key（例如有分组、接口名端口等等），然后 key 和 invoker 关联，转成export，将export缓存到 DubboProtocol 的 exporterMap 中。

之后其实就是打开 Server ，RPC 肯定需要远程调用，这里我们用的是 NettyServer 来监听服务。

