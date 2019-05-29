# RedisDelayTask
采用redis来实现延迟任务工具类

/**
 * redis 延迟任务工具类
 *
 * demo:
 * // 注册事件处理器
 * let task = require('./RedisDelayTask').getInstance();
 *  
 * let event_name = "test";
 * task.register(event_name, function(publish_name, publish_params, message){
 *    console.log(publish_name + "||"+ publish_params + "失效回调处理")
 * });
 * // 添加事件
 * task.add(event_name, 111, 2);
 *
 *  OR
 *
 * task.regAndAdd(event_name, 111, 2, function(publish_name, publish_params, message){
 *    console.log(publish_name + "||"+ publish_params + "失效回调处理")
 * });
 */
