---
title: LeetCode日记
date: 2022-09-04 15:37:16
tags:
---
## 关于LeetCode评测的机制
2022年9月2日，天气忘记怎么样了，反正我在图书馆摸鱼的时候一时兴起想到了LeetCode第三题"无重复字符的最长子串"的算法，就随手写了。

不得不说很多时候还是要面向对象比较好写，所以定义了一个CharSet类，顺便就创建了一个charset对象。

### 蜜汁失败
我觉得我算法没问题了，所以拿默认测例跑了一下，顺便处理好边界情况。看着这边测试也没问题，我就点了提交。
结果在"bbbbb"这个测例莫名其妙给我报错，预计结果为1，但我的代码跑出来是0。因为对自定义测例的掌握还不太熟练，我脑子里跑了一下，觉得没问题。
为了排除评测机抽风的极小概率事件，我原样重新提交了一遍，结果还是没过。

等我折腾了几遍终于学会了手动设置测例，接着提交，发现"bbbbb"这个手动设置的测例正常输出了1的结果。我再次怀疑起是否存在在编译时宇宙粒子改变了评测机中的某几个bit的可能性，又交了一遍，结果证明：当前地球的辐射水平应该还是比较正常的。

### 原因分析（虚假的原因）
考虑到输出的结果0与我初始化变量的值一致，我怀疑是否是代码没有顺利进入循环。为此，我偷偷把初始化值改成了-1，
果然这个测例给我跑出了-1的结果。但我死活想不到为什么它不进循环。

### 解决的过程（应该是真实的原因）
在我不断检查代码的时候，我开始思考是否是因为定义对象的问题，毕竟这次是第一次在LeetCode里用面向对象写。

我意识到定义完类之后顺便创建的对象是在LeetCode的函数外面的，转而意识到这个对象是全局变量。我试着把对象的声明放进函数内部，然后就过了。

### 进一步分析
引出一个猜想，LeetCode的评测应当是在一次运行中多次调用其设置的函数接口来实现的。意即，两次调用间，全局变量不会被析构，这在我的算法中
就会影响最终的结果：每次计算前，我的对象是需要初始化的，这个初始化过程被我封装在构造函数中，只在对象创建时被调用。
"ddddd"可能是所有测例中的第二个，在正常运行完第一个测例后由于没有重置charset对象，影响了第二个测例的结果。

因此，在写题的时候还要考虑对象的生命周期问题，对这种情形，应当将对象的声明放在函数内部，使其在函数被调用时被构造，返回时被析构，从而执行构造函数中的初始化。

或者，其实也可以不把初始化放在构造函数中，而是另外定义一个初始化函数，在使用前调用初始化。

### 代码附上
- 问题代码
```cpp
class CharSet {
    int situ[128];
    public:
        CharSet(){
            for(int i = 0; i < 128; i++){
                situ[i] = -1;
            }
        }
        int AddInfo(char c, int si){
            int index = (int)c;
            int ret = situ[index];
            situ[index] = si;
            return ret;
        }
} charset;


class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int slen = s.length();
        int Max_Length = -1;
        int Current_Length = 0;
        int le = 0, ri = 0;
        int sig;
        //线性读取
        for(ri = 0; ri < slen; ri ++){
            sig = charset.AddInfo(s[ri], ri);
            if(sig >= le){
                le = sig + 1;
            }
            Current_Length = ri - le + 1;
            if(Current_Length >= Max_Length){
                Max_Length = Current_Length;
            }
        }
        return Max_Length;
    }
};
```
- 通过代码
```cpp
class CharSet {
    int situ[128];
    public:
        CharSet(){
            for(int i = 0; i < 128; i++){
                situ[i] = -1;
            }
        }
        int AddInfo(char c, int si){
            int index = (int)c;
            int ret = situ[index];
            situ[index] = si;
            return ret;
        }
};


class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int slen = s.length();
        int Max_Length = -1;
        int Current_Length = 0;
        int le = 0, ri = 0;
        int sig;
        //线性读取
        for(ri = 0; ri < slen; ri ++){
            CharSet charset;
            sig = charset.AddInfo(s[ri], ri);
            if(sig >= le){
                le = sig + 1;
            }
            Current_Length = ri - le + 1;
            if(Current_Length >= Max_Length){
                Max_Length = Current_Length;
            }
        }
        return Max_Length;
    }
};
```