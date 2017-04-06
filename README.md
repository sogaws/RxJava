# RxJava
1、观察者模式和异步。观察者（Observer），订阅（Subscriber），被观察者（Observable），但因为流式API调用风格，实际代码为被观察者监听观察者即Observable.Subscriber（Observer）。

2、创建被观察者

正常模式：
Observable switcher=Observable.create(new Observable.OnSubscribe<String>(){

            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("On");
                subscriber.onNext("Off");
                subscriber.onNext("On");
                subscriber.onNext("On");
                subscriber.onCompleted();
            }
        });
这是最正宗的写法，创建了一个开关类，产生了五个事件，分别是：开，关，开，开，结束。

偷懒模式1

Observable switcher=Observable.just("On","Off","On","On");

偷懒模式2

String [] kk={"On","Off","On","On"};
Observable switcher=Observable.from(kk);

偷懒模式是一种简便的写法，实际上也都是被观察者把那些信息"On","Off","On","On"，包装成onNext（"On"）这样的事件依次发给观察者，当然，它自己补上了onComplete()事件。

3、创建观察者

正常模式
 Subscriber light=new Subscriber<String>() {
            @Override
            public void onCompleted() {
                //被观察者的onCompleted()事件会走到这里;
                Log.d("DDDDDD","结束观察...\n");
            }

            @Override
            public void onError(Throwable e) {
                    //出现错误会调用这个方法
            }
            @Override
            public void onNext(String s) {
                //处理传过来的onNext事件
                Log.d("DDDDD","handle this---"+s)
            }
这也是比较常见的写法，创建了一个台灯类。

偷懒模式(非正式写法)

        Action1 light=new Action1<String>() {
                @Override
                public void call(String s) {
                    Log.d("DDDDD","handle this---"+s)
                }
            }
            
之所以说它是非正式写法，是因为Action1是一个单纯的人畜无害的接口，和Observer没有啥关系，只不过它可以当做观察者来使，专门处理onNext 事件，这是一种为了简便偷懒的写法。当然还有Action0，Action2,Action3...,0,1,2,3分别表示call()这个方法能接受几个参数。

4、正常的观察者、被观察者、订阅的使用关系

//创建被观察者，是事件传递的起点

Observable.just("On","Off","On","On")

        //这就是在传递过程中对事件进行过滤操作
        
         .filter(new Func1<String, Boolean>() {
                    @Override
                    public Boolean call(String s) {
                        return s！=null;
                    }
                })  
        //实现订阅
        .subscribe(
                //创建观察者，作为事件传递的终点处理事件 
                  new Subscriber<String>() {
                        @Override
                        public void onCompleted() {
                            Log.d("DDDDDD","结束观察...\n");
                        }

                        @Override
                        public void onError(Throwable e) {
                            //出现错误会调用这个方法
                        }
                        @Override
                        public void onNext(String s) {
                            //处理事件
                            Log.d("DDDDD","handle this---"+s)
                        }
        );
        
总结一下：
创建被观察者，产生事件
设置事件传递过程中的过滤，合并，变换等加工操作。
订阅一个观察者对象，实现事件最终的处理。
Tips: 当调用订阅操作（即调用Observable.subscribe()方法）的时候，被观察者才真正开始发出事件。
