---
title: CompletableFuture && ScheduledExecutorService handle timeout
excerpt: 利用invoke,CompletableFuture,ScheduledExecutorService实现批处理接口，并支持超时处理
date: 2021-08-07 13:53
categories:
- Java
tags:
- 反射
- 多线程
- 线程池
---

## 背景

需要实现一个接口返回多个接口方法的数据，通过请求参数反射调用接口方法，如果接口方法超时，则返回默认值。
使用completableFuture，可以实现异步超时， jdk9已经有原生的实现，但是在jdk8需要自己做类似下面的实现, 需要利用applyToEigther的特性。

## Show Code

``` java
    private static final ScheduledExecutorService scheduler = ExecutorsManager.getInstance().getScheduledExecutorService();
    
    public BatchResponeVo batch(BatchRequestDto batchRequestDto) throws ExecutionException, InterruptedException {

        List<CompletableFuture<BatchResponeVo>> futuresList = Lists.newLinkedList();

        batchRequestDto.forEach((k, v) -> {
            BatchUriParamDto req = JsonUtil.parseJsonUseJackson(JsonUtil.toJsonUseJackson(v), BatchUriParamDto.class);
            Map<?, ?> param = Objects.requireNonNull(req).getParam();
            String uri = req.getUri();

            BatchServiceEnum serviceEnum = BatchServiceEnum.getEnumByServiceAndMethod(uri);
            CompletableFuture<BatchResponeVo> query = CompletableFuture.supplyAsync(() -> {
                BatchResponeVo result = new BatchResponeVo();
                try {
                    if (Objects.isNull(serviceEnum)) {
                        result.put(k, new BatchDataVo("SystemError", "not required service method", null));
                        return result;
                    }

                    Class<?> serviceClass = Class.forName(serviceEnum.getService());
                    Class<?> paramClass = Class.forName(serviceEnum.getParam());
                    Method method = serviceClass.getMethod(serviceEnum.getMethod(), paramClass);

                    Object obj = JsonUtil.parseJsonUseJackson(JsonUtil.toJsonUseJackson(param), paramClass);
                    Object bean = SpringUtil.getBean(serviceClass);
                    Object invoke = method.invoke(bean, obj);
                    result.put(k, new BatchDataVo("ok", null, invoke));
                    return result;
                } catch (ClassNotFoundException | InvocationTargetException | NoSuchMethodException |
                         IllegalAccessException e) {
                    log.error("ObservabilityService batch method error msg:{}", e.getMessage(), e);
                    result.put(k, new BatchDataVo("SystemError", e.getCause().getMessage(), null));
                    return result;
                }
            }, scheduler);

            final CompletableFuture<BatchResponeVo> chains = within(query, Duration.ofSeconds(10), k);
            futuresList.add(chains);
        });
        CompletableFuture<Void> allCompletableFuture = CompletableFuture.allOf(futuresList.toArray(new CompletableFuture[0]));

        BatchResponeVo result = new BatchResponeVo();
        allCompletableFuture.thenApply(v -> futuresList.stream().map(CompletableFuture::join).collect(Collectors.toList()))
                .get()
                .forEach(result::putAll);
        return result;
    }

    public CompletableFuture<BatchResponeVo> failAfter(Duration duration, String key){
        /// need a schedular executor
        final CompletableFuture<BatchResponeVo> timer = new CompletableFuture<>();
        scheduler.schedule(()-> timer.complete(new BatchResponeVo() {{
            put(key, new BatchDataVo("SystemError", "method excute timeout "+duration.get(SECONDS)+"s", null));
        }}),duration.toMillis(), TimeUnit.MILLISECONDS);
        return timer;
    }

    public CompletableFuture<BatchResponeVo> within(CompletableFuture<BatchResponeVo> taskFuture, Duration duration, String key){
        CompletableFuture<BatchResponeVo> timeoutWatcher = failAfter(duration, key);
        return taskFuture.applyToEither(timeoutWatcher, Function.identity());
    }
```

## 参考文档
* [使用CompletableFuture](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581182447650)
* [反射调用方法](https://www.liaoxuefeng.com/wiki/1252599548343744/1264803678201760)
* [Java中使用CompletableFuture处理异步超时](https://developer.aliyun.com/article/200625)
