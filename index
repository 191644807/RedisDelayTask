/**
 * Created by zhangfei on 2019/5/29.
 */
const defConf = {host: "127.0.0.1", port:6379, password: "", no_ready_check: true,};
const redis = require('redis');
const _ = require('lodash');
let uuid = 1;

/**
 * redis 延迟任务工具类
 *
 * demo:
 * // 注册事件处理器
 * let task = require('./RedisDelayTask').getInstance();
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
class RedisDelayTask{
    constructor(opt){
        this.conf = opt || defConf;
        if (!this.conf.db) {
            this.conf.db = 0;
        }

        // 对象唯一标识符，用于区分订阅事件
        this.uuid = uuid++;
        this.publisher = redis.createClient(this.conf);
        this.publisher.send_command('config', ['set','notify-keyspace-events','Ex']);

        this.subscriber = redis.createClient(this.conf);
        this.expired_subKey = '__keyevent@'+this.conf.db+'__:expired';
        this.subscribe();

        this.handlers = new Map(); // 关注事件处理列表
    }

    // 订阅 keyevent notification 事件
    subscribe() {
        this.subscriber.subscribe(this.expired_subKey,() => {
            this.subscriber.on('message', ( channel, message) => {
                const {uuid, publish_name, publish_params} = this.getEventsKey(message);
                if (uuid != this.uuid) {
                    return;
                }

                console.log(`uuid:${uuid},publish_name:${publish_name},publish_params:${publish_params}`);
                const handler = this.handlers.get(publish_name);
                if (handler && _.isFunction(handler)) {
                    handler(publish_name, publish_params, message)
                } else {
                    console.error('未找到失效事件处理器[%s]。', message)
                }
            });
        });
    }

    // 注册处理事件  fn(publish_name, publish_params, message)
    register(publish_name, fn) {
        this.handlers.set(publish_name, fn);
    }

    // 检查是否注册事件
    checkRegister(publish_name) {
        if (!this.handlers.has(publish_name)) {
           throw Exception(`This is ${publish_name} not register!`);
        }
    }

    /**
     * 添加处理事件
     * @param publish_name  发布主题名 string
     * @param publish_params  主题相关参数  string
     * @param delay  延迟处理时间 单位秒  int
     */
    add(publish_name, publish_params, delay = 10 * 60) {
        this.checkRegister(publish_name);
        const pubkey = this.getPubkey(publish_name, publish_params);
        this.publisher.setex(pubkey, delay, 'delay task event', (err) => {
            if (err) {
                return console.error('添加延迟事件失败：', err);
            }
            console.log(`添加延迟事件成功[${pubkey},${delay}]`);
        })
    }

    /**
     * 注册并添加事件
     * @param publish_name  发布时间的名称 string
     * @param publish_params 发布时间的参数 string
     * @param delay 延迟多少秒处理  单位秒 int
     * @param fn 任务延迟处理函数 function
     */
    regAndAdd(publish_name, publish_params, delay, fn){
        this.register(publish_name, fn);
        this.add(publish_name, publish_params, delay);
    }


    getPubkey(publish_name, publish_params){
        return `${this.uuid}:${publish_name}_${publish_params}`;
    }

    getEventsKey(message){
        let [uuid, events_key] = message.split(":");
        let [publish_name, publish_params] = events_key.split("_");
        return {uuid, publish_name, publish_params};
    }
}

// opt redis配置选项
module.exports = class RedisDelayTaskFactory{
    static new(opt){
        return new RedisDelayTask(opt);
    }

    static getInstance(opt){
        if (!RedisDelayTaskFactory.instance) {
            RedisDelayTaskFactory.instance = RedisDelayTaskFactory.new(opt);
        }
        return RedisDelayTaskFactory.instance;
    }
};
