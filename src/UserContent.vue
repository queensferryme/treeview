<template>
  <div
    id="user-content"
    ref="user-content"
    v-html="passage"
  >
  </div>
</template>

<script>
import marked from 'marked';

export default {
  name: 'UserContent',
  props: {
    active: { require: true },
    nodes: { require: true }
  },
  mounted () {
    this.$nextTick(() => {
      // initialize TOC tree
      let timeoutId = undefined;
      this.$emit('update:nodes', this.toc);
      console.log(this.passage);
      // scroll events
      this.$refs['user-content'].addEventListener('scroll', () => {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => {
          this.$emit('update:active', this.current.$path);
        }, 100);
      });
    });
  },
  data () {
    return {
      passage: marked("\
# 使用 Arduino 制作电子时钟\n\
## 设计思路\n\
### 面向对象  \n\
**优势**：将所有功能封装为对象的方法，逻辑清晰而易于管理\n\
```\n\
Datetime(int y, int m, int d, int h, int mi, int s) {\n\
  year = y;\n\
  month = m;\n\
  day = d;\n\
  hour = h;\n\
  minute = mi;\n\
  second = s;\n\
}\n\
```\n\
- 对象方法\n\
  - 时钟模式\n\
    - `void nexttick()`  \n\
    当前时间增加一秒\n\
    - `void show()`  \n\
    在 LCD 屏幕上打印出时间信息\n\
  - 修改模式\n\
    - `void increaseYear(int number)`\n\
    - `void increaseMont(int number)`\n\
    - `void increaseDay(int number)`\n\
    - `void increaseHour(int number)`\n\
    - `void increaseMinute(int number)`\n\
    - `void increaseSecond(int number)`\n\
  - 闹铃模式\n\
    - `Datetime operator =`\n\
    - `bool operator ==`\n\
\n\
### 业务逻辑  \n\
- 信号处理  \n\
![signal](/src/assets/signal.png)\n\
\n\
## 问题与调试\n\
### 利用字符串\n\
在使用`lcd.print()`显示时钟信息时，我们无法打印长度超过1的数字，但可以打印任意长度的字符串。所以只需要简单的将 `year`, `month`, `day`,` hour`, `minute`, `second` 等变量转化为字符串就能优雅地输出了。\n\
### F**k irrecv.resume()\n\
如果信号处理结束后没有调用`irrecv.resume()`方法重置接收器，会产生奇妙的效果。\n\
### 可是星期怎么办\n\
在我把 `year`, `month`, `day`,` hour`, `minute`, `second` 全部封装完毕以后，我发现我没有用来存储星期的变量。在增加一个变量很麻烦，所以我在使用了一个数学公式 —— 能从年月日计算出对应的星期\n\
```\n\
int weekday() {\n\
  int _y = year, _m = month, _d = day;\n\
  if (_m == 1 || _m == 2) {\n\
    _m += 12;\n\
    _y--;\n\
  }\n\
  return (_d + 1 + 2 * _m + 3 * (_m + 1) / 5 + _y + _y / 4 - _y / 100 + _y / 400) % 7;\n\
}\n\
```\n\
### 我想看看中间变量\n\
烧录到 arduino 板子上的程序运行情况究竟如何，这点是无法用肉眼观察的/doge。我们可以利用串口通信来将一些调试信息打印在串口监视器上。例如：`Serial.println(result.value, HEX);`\n\
## 实验结果\n\
一段小视频\n\
\n\
<video src='/src/assets/video.mp4' controls='controls' width='100%'></video>\n\
      ")
    }
  },
  computed: {
    current: {
      cache: false,
      get () {
        let res = this.headers[0];
        for (let elt of this.headers)
          if (elt.offsetTop <= this.$refs['user-content'].scrollTop)
            res = elt;
          else break;
        return res;
      }
    },
    headers () {
      return Array.from(this.$refs['user-content']
        .querySelectorAll('h1, h2, h3, h4, h5, h6'));
    },
    toc () {
      return this.headers.map((item) => {
          item.$value = item.innerText;
          item.$children = [];
          return item;
        }).reduce((result, current, index, array) => {
          let parent = (() => {
            for (let i = index - 1; i >= 0; i--)
              if (array[i].tagName < current.tagName)
                return array[i];
            return current;
          })();
          if (current !== parent) {
            parent.$children.push(current);
            current.$path = parent.$path.concat(parent.$children.length - 1);
          }
          else {
            result.push(current);
            current.$path = [0].concat(result.length - 1);
          }
          return result;
        }, []);
    }
  },
  watch: {
    'active' (next, prev) {
      let nextElt = this.toc[next[1]];
      for (let i = 2; i < next.length; i++)
        nextElt = nextElt.$children[next[i]];
      let prevElt = this.toc[prev[1]];
      for (let i = 2; i < prev.length; i++)
        prevElt = prevElt.$children[prev[i]];
      let scrollTop = this.$refs['user-content'].scrollTop;
      if ((nextElt.offsetTop > prevElt.offsetTop &&
          nextElt.offsetTop > scrollTop) ||
          (nextElt.offsetTop < prevElt.offsetTop &&
          nextElt.offsetTop < scrollTop &&
          nextElt !== this.current))
        this.$refs['user-content'].scrollTo({
          top: nextElt.offsetTop,
          behavior: 'smooth'
        });
    }
  }
}
</script>

<style scoped>
#user-content {
  overflow: auto;
  padding: 0 1rem;
}
#user-content >>> h1, #user-content >>> h2,
#user-content >>> h3, #user-content >>> h4,
#user-content >>> h5, #user-content >>> h6 {
  color: #01bb16;
}
#user-content >>> pre {
  background-color: #d6f5d6;
  border: 1px solid gray;
  border-radius: .3rem;
  font-family: 'Source Code Pro for powerline', sans-serif;
  font-size: 1.1rem;
  padding: 1% 3%;
}
#user-content >>> code {
  font-family: 'Source Code Pro for powerline', sans-serif;
  font-size: 1.1rem;
}
</style>
