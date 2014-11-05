---
layout: post
title:  使用纯JS播放音乐
date:   2014-11-01
summary: 教你如何使用Web audio API，并用纯js来写一个简单的音符播放器
---

本文示例页面:[http://blog.maxee.info/simplesheetmusic/example_cn.html](http://blog.maxee.info/simplesheetmusic/example_cn.html)  

源代码见[Github:ee0703/SimpleSheetMusic.js](https://github.com/ee0703/SimpleSheetMusic.js)

### 先看看原理 ###
先看维基百科的定义：  

>声音是通过物体振动产生的声波。是通过介质（空气或固体、液体）传播并能被人或动物听觉器官所感知的波动现象。
>声音的频率一般会以赫兹表示，记为Hz，指每秒钟周期性变化的次数。而分贝是用来表示声音强度的单位，记为dB。

HTML5带有Web Audio API，你可以使用AudioContext来创造声波, **webkit&firefox**内核浏览器都是支持的，但万恶的IE好像并不支持。

### Hello World ###
现在让我们来写我们的 Hello World：
{% highlight javascript %}
/*  创建AudioContext对象,
	firefox => AudioContext, 
	webkit => webkitAudioContext
*/
var context = new (window.AudioContext||window.webkitAudioContext)();

//创建波形合成器
oscillator = context.createOscillator();

//连接硬件
oscillator.connect(context.destination);
//开始播放声音
oscillator.start(0); 
{% endhighlight %}

把上面这段代码存成"music.js"，嵌入HTML中，用Chrome或Firefox试试。Very good，现在我们终于可以发音了，但是一直持续一个音符，非常很难听，简直就是噪音啊

### 旋律&音高 ###
**旋律**，就是**音高**的变化，音高，其实就是 **频率**，国际上，以中音'la'(也就是A4)的音高440HZ作为标准音。Allright，我们现在只要改变声波的频率，就能播放旋律了!让我们来修改代码:

{% highlight javascript %}
//AudioContext对象
var context = new (window.AudioContext||window.webkitAudioContext)();
//创建波形合成器
oscillator = context.createOscillator();

//连接硬件
oscillator.connect(context.destination);
//开始播放声音
oscillator.start(0); 

//每隔500毫秒改变一次音高
var t1 = setInterval(function(){
	//产生一个0~1800的随机频率(hz)
  	oscillator.frequency.value = (Math.random() *  1800)  
},
500);
{% endhighlight %}

下面是国际标准音高频率对照表（单位为hz）：

{% highlight javascript %}
/**
 * Equal Temperament Tuning
 * Source: http://www.phy.mtu.edu/~suits/notefreqs.html
 */
var tune_table = {
    'C0': 16.35,
    'C#0': 17.32,
    'Db0': 17.32,
    'D0': 18.35,
    'D#0': 19.45,
    'Eb0': 19.45,
    'E0': 20.60,
    'F0': 21.83,
    'F#0': 23.12,
    'Gb0': 23.12,
    'G0': 24.50,
    'G#0': 25.96,
    'Ab0': 25.96,
    'A0': 27.50,
    'A#0': 29.14,
    'Bb0': 29.14,
    'B0': 30.87,
    'C1': 32.70,
    'C#1': 34.65,
    'Db1': 34.65,
    'D1': 36.71,
    'D#1': 38.89,
    'Eb1': 38.89,
    'E1': 41.20,
    'F1': 43.65,
    'F#1': 46.25,
    'Gb1': 46.25,
    'G1': 49.00,
    'G#1': 51.91,
    'Ab1': 51.91,
    'A1': 55.00,
    'A#1': 58.27,
    'Bb1': 58.27,
    'B1': 61.74,
    'C2': 65.41,
    'C#2': 69.30,
    'Db2': 69.30,
    'D2': 73.42,
    'D#2': 77.78,
    'Eb2': 77.78,
    'E2': 82.41,
    'F2': 87.31,
    'F#2': 92.50,
    'Gb2': 92.50,
    'G2': 98.00,
    'G#2': 103.83,
    'Ab2': 103.83,
    'A2': 110.00,
    'A#2': 116.54,
    'Bb2': 116.54,
    'B2': 123.47,
    'C3': 130.81,
    'C#3': 138.59,
    'Db3': 138.59,
    'D3': 146.83,
    'D#3': 155.56,
    'Eb3': 155.56,
    'E3': 164.81,
    'F3': 174.61,
    'F#3': 185.00,
    'Gb3': 185.00,
    'G3': 196.00,
    'G#3': 207.65,
    'Ab3': 207.65,
    'A3': 220.00,
    'A#3': 233.08,
    'Bb3': 233.08,
    'B3': 246.94,
    'C4': 261.63,
    'C#4': 277.18,
    'Db4': 277.18,
    'D4': 293.66,
    'D#4': 311.13,
    'Eb4': 311.13,
    'E4': 329.63,
    'F4': 349.23,
    'F#4': 369.99,
    'Gb4': 369.99,
    'G4': 392.00,
    'G#4': 415.30,
    'Ab4': 415.30,
    'A4': 440.00,
    'A#4': 466.16,
    'Bb4': 466.16,
    'B4': 493.88,
    'C5': 523.25,
    'C#5': 554.37,
    'Db5': 554.37,
    'D5': 587.33,
    'D#5': 622.25,
    'Eb5': 622.25,
    'E5': 659.26,
    'F5': 698.46,
    'F#5': 739.99,
    'Gb5': 739.99,
    'G5': 783.99,
    'G#5': 830.61,
    'Ab5': 830.61,
    'A5': 880.00,
    'A#5': 932.33,
    'Bb5': 932.33,
    'B5': 987.77,
    'C6': 1046.50,
    'C#6': 1108.73,
    'Db6': 1108.73,
    'D6': 1174.66,
    'D#6': 1244.51,
    'Eb6': 1244.51,
    'E6': 1318.51,
    'F6': 1396.91,
    'F#6': 1479.98,
    'Gb6': 1479.98,
    'G6': 1567.98,
    'G#6': 1661.22,
    'Ab6': 1661.22,
    'A6': 1760.00,
    'A#6': 1864.66,
    'Bb6': 1864.66,
    'B6': 1975.53,
    'C7': 2093.00,
    'C#7': 2217.46,
    'Db7': 2217.46,
    'D7': 2349.32,
    'D#7': 2489.02,
    'Eb7': 2489.02,
    'E7': 2637.02,
    'F7': 2793.83,
    'F#7': 2959.96,
    'Gb7': 2959.96,
    'G7': 3135.96,
    'G#7': 3322.44,
    'Ab7': 3322.44,
    'A7': 3520.00,
    'A#7': 3729.31,
    'Bb7': 3729.31,
    'B7': 3951.07,
    'C8': 4186.01
	};
{% endhighlight %}

### 节奏&音符 ###
**节奏**是音乐的骨架，上面的例子其实已经包含节奏了，我们的定时器500ms执行一次，你可以想象一下，类似于每隔500ms我们就拍一次巴掌。也就是每分钟要拍120次，这个120，这就是我们的节拍速度，每拍一次掌，就是一次**拍子**。  
有了节奏，我们就可以用符号来表示音乐了，因为一切都在节拍的骨架里。**音符**，就是用来表示在这种节拍框架下，每个音的符号。很容易理解，**音符=音高+持续时间**，比如,'do',持续四个拍子。  
知道这些以后，我们就可以演奏我们的音乐啦，比如我们要唱'do re mi fa so la xi do'，是这样的：

{% highlight javascript %}
[C4(do) 1拍] [D4(re) 1拍] [E4(mi) 1拍] [F4(fa) 1拍]
[G4(so) 1拍] [A4(la) 1拍] [B4(xi) 1拍] [C5(do) 1拍]
{% endhighlight %}

现在，假定1拍为500ms，我们来play it：
	
{% highlight javascript %}
//创建oscillator
var context = new (window.AudioContext||window.webkitAudioContext)(),
oscillator = context.createOscillator();
oscillator.connect(context.destination);
oscillator.start(0);

//为了方便起见，这里只包含我们用到的音符频率
notes_table = {
    'C4': 261.63,
    'D4': 293.66,
    'E4': 329.63,
    'F4': 349.23,
    'G4': 392.00,
    'A4': 440.00,
    'B4': 493.88,
    'C5': 523.25,
};
	
//歌曲的音符序列
var song = [['C4',1], ['D4',1], ['E4',1], ['F4',1],
			 ['G4',1], ['A4',1], ['B4',1], ['C5',1]];
var i = 0;
//演奏歌曲
(function(){
  		if(i < song.length) {
  			//频率
  			oscillator.frequency.value = notes_table[song[i][0]];
  			//持续时间（ms)
  			var last = song[i][1] * 500;
  			i++;
  			//音符持续时间后，调用自己进行下一个音符
  			setTimeout(arguments.callee,last);
  		}
 })();
{% endhighlight %}

同样的旋律，节奏改变了，就是另一段音乐了，现在我们来试试另一种节奏，把我们的歌曲改成:
	
{% highlight javascript %}
var song =[['C4',2], ['D4',1], ['E4',2], ['F4',1],
		 	['G4',2], ['A4',1], ['B4',2], ['C5',1]];
{% endhighlight %}

怎么样，是不是风格大不一样了?

### 停止&休止符 ###

现在，问题来了，我们的音乐演奏完后，并没有停止，这让人很不爽。其实安静也是音乐中的部分，有时候，我们在也音乐中也会停止几个节拍，这种停止音符称为**休止符**。  
我们可以调用 oscillator.stop() 来停止我们的音乐，把播放函数改成:

{% highlight javascript %}
(function(){
	if(i < song.length) {
  		//频率
  		oscillator.frequency.value = notes_table[song[i][0]];
  		//持续时间
  		var last = song[i][1]*500;
  		i++;
  		//音符持续时间后，调用自己进行下一个音符
  		setTimeout(arguments.callee,last);
  	}else{
		 //停止播放
  		 oscillator.stop()
  	}
 })();
{% endhighlight %}

很好，现在音乐结束后，可以停下来了，但是仍然不支持休止符，假设用'0'表示休止符，遇到休止符我们stop就行了(特别注意:stop以后，需要重新创建oscillator对象)。为了方便起见，我们把现有代码重构一下，封装我们的播放simple_player对象:  

{% highlight javascript %}
//为了方便起见，这里只包含我们用到的音符频率
notes_table = {
    'C4': 261.63,
    'D4': 293.66,
    'E4': 329.63,
    'F4': 349.23,
    'G4': 392.00,
    'A4': 440.00,
    'B4': 493.88,
    'C5': 523.25,
};
//播放器对象
function simple_player(){
	//播放状态
	this.status = 0;
	//速度(拍/每分钟)
	this.tempo = 120;
	//新建oscillator
	this.new_oscillator = function(){
    	//AudioContext对象
	    this.context = new (window.AudioContext || window.webkitAudioContext)();
	    //oscillator对象
	    this.oscillator = this.context.createOscillator();
	    //连接到硬件
	    this.oscillator.connect(this.context.destination);
    }
	
	//私有函数--播放音乐
	function play_sheet_music(sheet_music, player_obj){
	    var i = 0;
		var interval_time = 60000 / player_obj.tempo;
		(function(){
	  		if(i < sheet_music.length) {
	  			//音符
	  			var music_note = sheet_music[i][0];
	  			//持续时间
	  			var last = sheet_music[i][1]*interval_time;

	  			if(music_note == '0'){
	  				//处理休止符
	  				if(player_obj.status && player_obj.oscillator){
	  					player_obj.oscillator.stop(0);
	  					player_obj.status = 0;
	  				} 
	  			}else{
					//正常音符
	  				if(!player_obj.status){
	  					player_obj.status = 1;
	  					//新建oscillator
	  					player_obj.new_oscillator();
	  					player_obj.oscillator.start(0);
	  				}
	  				//更改频率
	  				player_obj.oscillator.frequency.value = notes_table[music_note];
	  			}

	  			i++;
	  			//音符持续时间后，调用自己进行下一个音符
	  			setTimeout(arguments.callee,last,sheet_music,player_obj);
	  		}else{
	  			 //停止播放
	  			 player_obj.oscillator.stop(0);
	  		}
	 })();
	}

	//播放音乐，对外接口
	this.play = function(sheet_music){
		play_sheet_music(sheet_music,this);
	}
}
{% endhighlight %}

这样，player就可以很方便的使用了，我们来测试一下休止符:

{% highlight javascript %}
//音符序列
var song = [['C4',1], ['D4',1], ['E4',1],['0',1],
 			['F4',1],['G4',2], ['A4',1], ['0',1],
 			['B4',2], ['C5',1]];

//播放
var player1 = new simple_player();
player1.play(song);
{% endhighlight %}

嗯，perfect，但还是有问题，让我们试试播放《小星星》:  

{% highlight javascript %}
//小星星: 1155665- 4433221-
var song = [['C4',1], ['C4',1], ['G4',1],['G4',1],['A4',1],['A4',1],['G4',2], 
			['F4',1], ['F4',1], ['E4',1],['E4',1],['D4',1],['D4',1],['C4',2]];

//播放
var player1 = new simple_player();
player1.play(song);
{% endhighlight %}

两个相同的音符被连在一起了!"do do so so la la so" 变成了"do- so- la- so"，非常糟糕，要解决这一点，我们需要从音量下手。  

### 音量 ###
**音量**，其实就是声波的振动幅度，注意我们的每一个音符音量并不是恒定的，有一个**淡入淡出效果**。要控制音量，我们可以使用 GainNode，关于GainNode的用法可以[参考这里](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext.createGain)。  
以下修改player代码为为我们的音符加入淡出(实际上只有淡出，没有淡入):  


{% highlight javascript %}
//播放器对象
function simple_player(){
	//播放状态
	this.status = 0;
	//速度(拍/每分钟)
	this.tempo = 120;
	//播放音量 0.0 ~ 1.0 
	this.volume = 0.7;
	//新建oscillator
	this.new_oscillator = function(){
		//AudioContext对象
		this.context = new (window.AudioContext || window.webkitAudioContext)();
		//oscillator对象
		this.oscillator = this.context.createOscillator();
		//声音大小
		this.volumeNode = this.context.createGain();
		//连接到硬件
		this.volumeNode.connect(this.context.destination);
		this.oscillator.connect(this.volumeNode);
	}

	//私有函数--音量淡出
	function fadeout(volume_node,volume,fade_time)
	{
		volume_node.gain.value = volume;
		var i = 0;
		var intvl = setInterval(function(){	
				//对后半段进行指数衰变
				if(i < 9)
					volume_node.gain.value =  volume_node.gain.value * 0.92 ; 
					//注:你也用正弦或其他函数,如: volume*Math.sin ...
				else
					clearInterval(intvl);
				i++;
			},(fade_time/10));
	}

	//私有函数--播放音乐
	function play_sheet_music(sheet_music, player_obj){
		var i = 0;
		var interval_time = 60000 / player_obj.tempo;
		(function(){
	  		if(i < sheet_music.length) {
	  			//音符
	  			var music_note = sheet_music[i][0];
	  			//持续时间
	  			var last = sheet_music[i][1]*interval_time;

	  			if(music_note == '0'){
	  				//处理休止符
	  				if(player_obj.status && player_obj.oscillator){
	  					player_obj.oscillator.stop(0);
	  					player_obj.status = 0;
	  				} 
	  			}else{
					//正常音符
	  				if(!player_obj.status){
	  					player_obj.status = 1;
	  					//新建oscillator
	  					player_obj.new_oscillator();
	  					player_obj.oscillator.start(0);
	  				}
	  				//更改频率
	  				player_obj.oscillator.frequency.value = notes_table[music_note];
	  			}
	  			i++;
  				//音量淡出效果
  				fadeout(player_obj.volumeNode,player_obj.volume,last);
	  			//音符持续时间后，调用自己进行下一个音符
	  			setTimeout(arguments.callee,last,sheet_music,player_obj);
	  		}else{
	  			 //停止播放
	  			 player_obj.oscillator.stop(0);
	  		}
	 })();
	}

	//播放音乐，对外接口
	this.play = function(sheet_music){
		play_sheet_music(sheet_music,this);
	}
}
{% endhighlight %}

再试试用它来播放《小星星》，好像有淡出效果了！但是音色非常单调啊，下面让我们来看看音色是什么。 

### 音色 ###
其实刚才的声音只有一种频率的波形，那就是**基音**频率的波形。现实的声波组成要复杂的多，比如用钢琴弹奏一个标准音la(440HZ)，并不是只有440HZ的波形，同时会产生一系列各种频率的波，但是频率为440HZ的波对音高起到决定作用，我们称为**基音**。除了基音以外，其他一系列频率的波，我们称为 **泛音**，一般而言， **基音决定音高，泛音决定音色**。  
即使是基音，也是由多种基本波形组成的，你可以试试改变波形，看看音色有什么变化:  


{% highlight javascript %}
//AudioContext对象
var context = new (window.AudioContext || window.webkitAudioContext)();
oscillator = context.createOscillator();
oscillator.connect(context.destination);
oscillator.start(0); 

//试试以下这些波形
oscillator.type = 'sine'    	//正弦波，最常见，默认值
oscillator.type = 'square'  	//方波
oscillator.type = 'sawtooth'    //锯齿波
oscillator.type = 'triangle'    //三角波

{% endhighlight %}

钢琴和吉他演奏同一个音符，音色区别很大，是因为他们的泛音序列不一样。由于音色特别复杂，本人也不太懂，所以这里就不再往下说了，感兴趣的话，你可以自己试试给主音加上泛音，相关API可以参考W3C的[Web Audio API](https://dvcs.w3.org/hg/audio/raw-file/tip/webaudio/specification.html)。另外，作为音色的科普，你可以读一读[音色的度娘百科](http://baike.baidu.com/view/21387.htm?fr=aladdin)。
