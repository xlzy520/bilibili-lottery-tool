# bilibili-lottery-tool
哔哩哔哩(bilibili)评论区抽奖工具，超硬核，可以去重，查看抽奖的人是否关注自己

### 使用方式
1. Chrome、Edge、Firefox浏览器打开想要抽奖的视频页面，例如：https://www.bilibili.com/video/BV13K4y1X7dC
2. F12或者点击右键》选择检查，打开控制台，选择console【中文：控制台】这个标签页
3. 把下面代码粘贴进去，然后按回车即可, **最后一行有个数字2，代表抽几个人，改成几就抽几个**

#### 代码
```js
class Lottery {
  constructor(num) {
    this._oid = window.__INITIAL_STATE__.aid;
    this._number = num;
  }

  async getLuckyUsers() {
    try {
      const count = await this.getCommentsCount()
      const ps = Math.ceil(count / 30)
      const promises = Array(ps).fill('1').map((value, index) => {
        return this.getComments(index)
      })
      const allData = await Promise.all(promises)
      const replies = []
      const replyMapping = {}
      allData.forEach(arr=> {
        arr.forEach(v=> {
          if (!replyMapping[v.mid]) {
            replyMapping[v.mid] = v
            replies.push(v)
          }
        })
      })
      console.log(">>>>>> 正在获取评论列表 <<<<<<");
      console.log("评论列表:", replies.map(v=> ({
        name: v.member.uname,
        content: v.content.message
      })));
      console.log(">>>>>> 正在选取幸运用户 <<<<<<");
      const _arr = this.shuffleSelf(replies)
      const lucky = []
      for (let i = 0; i < this._number; i++) {
        lucky.push(_arr.pop())
      }
      const allRelations = lucky.map(v=> this.getRelations(v.mid))
      const relations = await Promise.all(allRelations)
      const map = {
        0: '未关注',
        2: '已关注',
        6: '互相关注',
        undefined: '我自己啦'
      }
      console.group('恭喜中奖用户：')
      lucky.forEach((item, index) => {
        item.relationText = map[relations[index].attribute]
      })
      
      console.table(lucky.map(v=> ({
        用户名: v.member.uname,
        uid: v.member.mid,
        内容: v.content.message,
        是否关注: v.relationText,
      })))
      console.groupEnd('恭喜中奖用户：')
    } catch (e) {
      console.log(e);
    }
  }
  
  shuffleSelf(array, size) {
    var index = -1,
      length = array.length,
      lastIndex = length - 1;
    size = size === undefined ? length : size;
    while (++index < size) {
      var rand = index + Math.floor( Math.random() * (lastIndex - index + 1)),
      value = array[rand];
      
      array[rand] = array[index];
      
      array[index] = value;
    }
    array.length = size;
    return array;
  }
  
  getCommentsCount(next = 0) {
    return fetch(`https://api.bilibili.com/x/v2/reply/main?jsonp=jsonp&ps=30&next=${next}&type=1&oid=${this._oid}`)
      .then(res => {
        return res.json()
      }).then(res => {
        return res.data.cursor.all_count
      })
  }
  
  
  getComments(next = 0) {
    return fetch(`https://api.bilibili.com/x/v2/reply/main?jsonp=jsonp&ps=30&next=${next}&type=1&oid=${this._oid}`)
      .then(res => {
        return res.json()
      }).then(res => {
        return res.data.replies || []
      })
  }
  
  getRelations(uid){
    return fetch('https://api.bilibili.com/x/space/acc/relation?mid='+uid, {
      credentials: "include"
    }).then(res => {
      return res.json()
    }).then(res => {
      return res.data.be_relation
    })
  }
}

new Lottery(2).getLuckyUsers();
```
