##使用Mob进行短信验证

 <ul>
 <li>1、准备工作</li>
 <li>2、工程配置</li>
 <li>3、短息验证码获取</li>
 <li>4、语音验证码</li>
 <li>5、UI的处理</li>
 </ul>


<p> 一、准备工作

    准备工作主要是在Mob页面进行的工作，包括以下：
       （1）首先去Mob官网下载SMS for Android 的SDK
       （2）注册账号，登录进去
       （3）进入后台，创建Andorid 应用（应用名要和Android studio中module名相同）。
       （4）记住自己的app Key 和 app Secret



<p> 二、工程配置

    在Android studio中配置mob的环境，很简单：
        （1）将下载下来的SDK文件解压出来
        （2）将SMSSDK文件夹下的aar文件和jar文件拷贝到相应module的libs文件夹下
            如果不使用mob提供的页面，可以不拷贝SNSSDKGUI的jar文件
        （3）打开module的build.gradle文件，
            添加
            =================================================
                repositories{
                    flatDir{
                        dirs 'libs' //就是你放aar的目录地址
                    }
                }
                
            在dependencies中添加
            ==================================================
            compile name:'SMSSDK-2.1.3',ext:'aar'   //注意这里的版本号换成你自己的版本号
            //compile name:'SMSSDKGUI-<version>',ext:'aar' //这是对GUI的依赖，如果没有添加gui的jar包，则不添加这一行
            
            
然后，sync一下整个工程，就可以进行mob的短信验证开发了        
            
            
            
<p> 三、短信验证码的获取          
            
    （1）首先在工程中对Mob进行初始化声明：
           可以在onCreate函数中添加
            ==================================================
            SMSSDK.initSDK(this,appKey,appSecret);
            SMSSDK.registerEventHandler(eventHandler);// 注册短信回调，
            
            注意务必在onDestroy中取消短信注册
            ==================================================
            SMSSDK.unregisterEventHandler(eventHandler);// 取消注册短信回调
            
            
    (2)在上边一步组册短信回调时有一个 eventHandler ，这个参数就是需要我们来实现的接受Mob短信回调要用的
            可以这个声明：
            ==================================================
            private EventHandler eventHandler=new EventHandler(){
                    @Override
                    public void afterEvent(int event, int result, Object o) {
                        if (result==SMSSDK.RESULT_COMPLETE){
                            if (event==SMSSDK.EVENT_SUBMIT_VERIFICATION_CODE){//提交验证码成功
                                Log.i(TAG, "提交验证码成功");
                            }else if (event==SMSSDK.EVENT_GET_VERIFICATION_CODE){//获取验证码成功
                                Log.i(TAG, "获取验证码成功");
                            }else if (event==SMSSDK.EVENT_GET_SUPPORTED_COUNTRIES){//返回支持发送验证码的国家列表
                                Log.i(TAG, "返回支持发送验证码的国家列表");
                            }else if(event==SMSSDK.EVENT_GET_VOICE_VERIFICATION_CODE){
                                Log.i(TAG, "返回语音播报");//返回的是包含phone和国家号的json
                            }else {Log.d(TAG, "验证失败");
                                ((Throwable) o).printStackTrace();
                            }
                        }
            
                    }
                }; //用来接受回调的mob短信信息
                
                
                无论是否验证成功，对接下来的操作不建议在该ecentHandler中直接进行，建议新建一个Handler来处理相应的操作
                ==================================================
                代码不再详贴
                
    （3）至此接收流程已经完成，就差一个触发事件了，
            可以对不同的按钮添加点击监听，在监听函数中分别添加
            ==================================================
            getSupportedCountries();//获取所支持 手机号的国家地区
            getVerificationCode("86", phone);//向手机为phone的号码发送验证码
            submitVerificationCode("86", phone, code);//将手机号与验证码结合起来进行验证，通过即可进行接下来的活动
                
            
            
<p> 四、语音验证码的获取         

    其实语音验证码完全可以放在第三步的第（3）小节中，甚至更加简单一些。
            语音验证，只需要在相应的触发事件里添加：
            =================================================
            getVoiceVerifyCode("86", phone);
            
            就可以了，也没必要在eventHanlder中添加对该消息的接收处理。
            因为本函数调用后，会收到一个播报验证码的电话。要做的就是保证在接通电话时本app不会被杀死就行。
            
            
        

<p> 五、UI的处理
        
        UI的处理，有很多细节需要注意，包括上边所说的创建Handler来处理相应的UI操作。这种不再赘述。
        这里来说一下常见的一个小场景：
        
            当点击完获取验证码后，该Button会编程不可点击状态，且其内的文字会变成倒计时状态显示
            
        实现起来,并不难：
        
        首先来设置Button不可点击
        =================================================
        xml中的设置方式是： android:enabled="false"
        javad代码设置的方式是：btn.setEnable(false);
        
        
        然后就可以在初次点击后对该btn添加倒计时状态了
        =================================================
        private void changBtnState() {
                Thread thread=new Thread(){
                    @Override
                    public void run() {
                        super.run();
                        if (changing){
                            while(mTime>0){
                                mTime--;
                                if (PhoneLoginActivity.this==null){
                                    break;
                                }
        
                                PhoneLoginActivity.this.runOnUiThread(new Runnable() {
                                    @Override
                                    public void run() {
                                        btn_get_vertify.setText("获取中("+mTime+")");
                                        btn_get_vertify.setEnabled(false);
                                    }
                                });
        
                                try {
                                    Thread.sleep(1000);
                                } catch (InterruptedException e) {
                                    e.printStackTrace();
                                }
                            }
                            changing=false;
                        }
        
        
                        mTime=60;
                        changing=true;
        
        
                        if (PhoneLoginActivity.this!=null){
                            PhoneLoginActivity.this.runOnUiThread(new Runnable() {
                                @Override
                                public void run() {
                                    btn_get_vertify.setText("获取验证码");
                                    btn_get_vertify.setEnabled(true);
                                }
                            });
                        }
                    }
                };
        
                thread.start();
            }
        
        


<p>到此，功能的骨干任务已经完成，具体代码可以参考工程。


