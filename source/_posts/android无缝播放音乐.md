---
title: android端音乐播放(无缝)
---


*-by 李泽君 2019-10-25*

- #### 项目背景: 眼罩蓝牙项目
- #### 实现功能: 音乐无缝播放及混合播放

---
前言
> MediaPlayer 是多媒体包的一个 基本工具，但它一次只能处理一个音频或 视频文件.如果只有少量的音频要播放，并且想要快速的性能，SoundPool 类可提供帮助, 它在底层使用了 MediaPlayer APISoundPool 与 MediaPlayer 之间的一个区别是:  SoundPool 是仅为处理本地媒体文件而设计的

------------


>11-20更新 **setNextMediaPlayer实现方式**

```
private int mPlayResId = R.raw.wav1;
    public void testLoopPlayer() {
        mediaPlayer = MediaPlayer.create(this, mPlayResId);
        mediaPlayer.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
            @Override
            public void onPrepared(MediaPlayer mediaPlayer) {
                mediaPlayer.start();
            }
        });
        createNextMediaPlayer();
    }

    private void createNextMediaPlayer() {
        nMediaPlayer = MediaPlayer.create(this, mPlayResId);
        mediaPlayer.setNextMediaPlayer(nMediaPlayer);
        mediaPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
            @Override
            public void onCompletion(MediaPlayer mp) {
                mp.release();

                mediaPlayer = nMediaPlayer;

                createNextMediaPlayer();
            }
        });
    }
```


------------


#### 一, MediaPlayer实现

######MediaPlayer简介:
Android MediaPlayer 设置Loop=true之后呢音乐是会循环，但循环中间会出现停顿，其实原来就是再整首播放结束后再调用MediaPlayer的start函数，但start函数调用是需要时间，这时间就是停顿的原因.

基本思路:
设置两个播放器, 在第一个播放时, 设置监听器, 对进度进行判断, 若到结尾, 则启动第二个播放器

```
		
		mediaPlayerOne = new MediaPlayer();
        mediaPlayerTwo = new MediaPlayer();

        AssetManager assetManager = getAssets();
        AssetFileDescriptor descriptor = null;

        try {
            descriptor = assetManager.openFd("3dmbridge.wav");

            mediaPlayerOne.setDataSource(descriptor.getFileDescriptor(), descriptor.getStartOffset(), descriptor.getLength());
            mediaPlayerTwo.setDataSource(descriptor.getFileDescriptor(), descriptor.getStartOffset(), descriptor.getLength());
            mediaPlayerOne.prepareAsync();
            mediaPlayerTwo.prepareAsync();
        } catch (IOException e) {
            e.printStackTrace();
        }

        viewById.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                mediaPlayerOne.start();
                setDownTimerByTime(24 * 60 * 60 * 1000, 10);
            }
        });
    }


    public void setDownTimerByTime(int totalTime,int time){
        if(timerSleep !=null){
            timerSleep.cancel();
        }
        timerSleep = new CountDownTimer(totalTime, time) {

            @Override
            public void onTick(long millisUntilFinished) {

                if(mediaplayIndex ==0){
                    //当前播放的是第一个播放器
                    if(mediaPlayerOne.isPlaying()) {
                        int av = mediaPlayerOne.getDuration() - mediaPlayerOne.getCurrentPosition();
                        if (av <= 5000) {
                            //如果播放快结束了，距离结束还剩下5秒，这个时候就要通过毫秒来获取进度了
                            setTimeMil(24 * 60 * 60 * 1000, 10);
                            if (timerSleep != null) {
                                timerSleep.cancel();
                            }
                        }
                    }
                }else{
                    //当前播放的是第二个播放器
                    if(mediaPlayerTwo.isPlaying()) {
                        int av = mediaPlayerTwo.getDuration() - mediaPlayerTwo.getCurrentPosition();
                        if (av <= 5000) {
                            setTimeMil(24 * 60 * 60 * 1000, 10);
                            if (timerSleep != null) {
                                timerSleep.cancel();
                            }
                        }
                    }

                }


            }
            @Override
            public void onFinish() {

            }
        }.start();

    }


    public void setTimeMil(int totalTime,int time){

        if(timerSleepMil !=null){
            timerSleepMil.cancel();
        }
        timerSleepMil = new CountDownTimer(totalTime, time) {

            @Override
            public void onTick(long millisUntilFinished) {

                if(mediaPlayerOne !=null){
                    if(mediaplayIndex==0){
                        if(mediaPlayerOne.isPlaying()) {
                            int av = mediaPlayerOne.getDuration() - mediaPlayerOne.getCurrentPosition();
                            if (av <= 250) {
                                //距离快结束250毫秒的时候，启动另外一个音乐播放器，但是音量不适合太大，不然会有重音，暂定0.25吧。
                                if (!mediaPlayerTwo.isPlaying()) {
                                    mediaPlayerTwo.setVolume(0.25f,0.25f);
                                    mediaPlayerTwo.start();
                                }
                            }

                            if(av<=250&&av>=100){
                                //在进度剩余100-250的时候呢，对第一个播放器进行降音，对第二播放器进行升音。
                                float ind=0.75f-((av*3)/1000f);
                                mediaPlayerOne.setVolume(1-ind,1-ind);
                                mediaPlayerTwo.setVolume(0.25f+ind,0.25f+ind);
                            }

                            if (av <= 100) {
                                //距离快结束100毫秒的时候，关掉当前播放的播放器，我也想不通我为什么要这么做。
                                if(mediaPlayerOne.isPlaying()) {
                                    mediaPlayerOne.seekTo(0);
                                }
                                if(mediaPlayerOne.isPlaying()) {
                                    mediaPlayerOne.pause();
                                }
                                mediaplayIndex = 1;

                                setDownTimerByTime(24 * 60 * 60 * 1000, 1000);
                                if (timerSleepMil != null) {
                                    timerSleepMil.cancel();
                                }
                            }
                        }

                    }else{
                        //同上面一致
                        if(mediaPlayerTwo.isPlaying()) {
                            int av = mediaPlayerTwo.getDuration() - mediaPlayerTwo.getCurrentPosition();
                            if (av <= 250) {
                                if (!mediaPlayerOne.isPlaying()) {
                                    mediaPlayerOne.setVolume(0.25f,0.25f);
                                    mediaPlayerOne.start();
                                }
                            }

                            if(av<=250&&av>=100){

                                float ind=0.75f-((av*3)/1000f);
                                mediaPlayerTwo.setVolume(1-ind,1-ind);
                                mediaPlayerOne.setVolume(0.25f+ind,0.25f+ind);
                            }

                            if (av <= 100) {
                                if(mediaPlayerTwo.isPlaying()) {
                                    mediaPlayerTwo.seekTo(0);
                                }
                                if(mediaPlayerTwo.isPlaying()) {
                                    mediaPlayerTwo.pause();
                                }
                                mediaplayIndex = 0;
                                setDownTimerByTime(24 * 60 * 60 * 1000, 1000);
                                if (timerSleepMil != null) {
                                    timerSleepMil.cancel();
                                }
                            }
                        }
                    }
                }

            }
            @Override
            public void onFinish() {

            }
        }.start();

    }
```
------------



>在使用MediaPlayer播放一段流媒体的时候，需要使用prepare()或prepareAsync()方法把流媒体装载进MediaPlayer，才可以调用start()方法播放流媒体。setAudioStreamType()方法用于指定播放流媒体的类型，它传递的是一个int类型的数据，均以常量定义在AudioManager类中， 一般我们播放音频文件，设置为AudioManager.STREAM_MUSIC即可

------------

注意释放操作:
```
	@Override
    protected void onDestroy() {
        super.onDestroy();
        if (mediaPlayer != null && mediaPlayer.isPlaying()) {
            mediaPlayer.stop();
            mediaPlayer.release();
            mediaPlayer = null;
        }
    }
```

#### 二 SoundPool实现:
######SoundPool简介:
SoundPool支持多个音频文件同时播放(组合音频也是有上限的)，延时短，比较适合短促、密集的场景

实现思路:
直接设置为循环播放即可:


```

		AssetManager assetManager = getAssets();
        AssetFileDescriptor descriptor = null;


		 final SoundPool soundPool = new SoundPool(4, 0, 0);
        final int haha = soundPool.load(descriptor, 1);
        soundPool.setOnLoadCompleteListener(new SoundPool.OnLoadCompleteListener() {
            @Override
            public void onLoadComplete(SoundPool soundPool, int i, int i1) {
                Log.d(TAG, "onLoadComplete: " + i + "...." + i1);
                Log.d(TAG, "soundPool play time: " + System.currentTimeMillis());
            }
        });
        button2.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Toast.makeText(view.getContext(), "bofang ", Toast.LENGTH_LONG).show();
                Log.d(TAG, "soundPool play time: " + System.currentTimeMillis());
                soundPool.play(haha, 0.5f, 0.5f, 1, -1, 1.0f);
            }
        });
```

>注意点:
``loop mode (0 = no loop, -1 = loop forever)``

音频播放的两种方式, 以后在项目中可以作为模板代码使用.












