# 网页版五子棋|GOBANG-N

## 1 运行项目

1. 直接用浏览器打开***public***文件夹下***index.html***文件即可。
2. 终端（控制台）输入`node index.js`启动一个静态文件服务器，然后在浏览器地址栏输入`http://127.0.0.1:8080/`即可打开网页。
3. **src**目录下为源码，不可直接在浏览器运行；**public**目录下是通过**webpack**打包之后生成的静态网页文件，可以直接在浏览器运行。



## 2 实现功能

- [x] 可改变棋盘大小
- [x] 可设获胜所需的棋子数
- [x] 悔棋
- [x] 棋盘调试
- [x] 贪心算法
- [x] 博弈树
- [x] alpha-beta减枝
- [x] 启发式(先验)减枝
- [x] 迭代深化
- [x] 禁手
- [x] 网页多线程
- [x] AI设置界面
- [x] 设置时限



## 3 界面展示

### 3.1 初始界面

<img src="https://dzy-typora-img-hosting-service.oss-cn-shanghai.aliyuncs.com/typoraImgs/image-20200901175002492.png" alt="image-20200901175002492" style="zoom:50%;" />

### 调整棋盘大小

<img src="https://dzy-typora-img-hosting-service.oss-cn-shanghai.aliyuncs.com/typoraImgs/web-2.png" alt="image-20200901175516537" style="zoom:50%;" />

### 3.2 调整胜利条件

可以玩1~6子棋

<img src="https://dzy-typora-img-hosting-service.oss-cn-shanghai.aliyuncs.com/typoraImgs/web-3.png" alt="image-20200901175604020" style="zoom:50%;" />

### 3.3 调整AI参数

- 递归深度：2-10层
- 搜索广度：2-24个
- 时间限制：0.1s-10s
- 是否允许禁手

<img src="https://dzy-typora-img-hosting-service.oss-cn-shanghai.aliyuncs.com/typoraImgs/web-4.png" alt="image-20200901175734822" style="zoom:50%;" />

### 3.4 切换调试模式（人人对战）

<img src="https://dzy-typora-img-hosting-service.oss-cn-shanghai.aliyuncs.com/typoraImgs/web-5.png" alt="image-20200901180058906" style="zoom:50%;" />

### 3.5 提示信息

每步棋标有序号，提示信息栏中有AI估值、思考时间、递归深度等信息。

<img src="https://dzy-typora-img-hosting-service.oss-cn-shanghai.aliyuncs.com/typoraImgs/image-20200901180210657.png" alt="image-20200901180210657" style="zoom:50%;" />

### 3.6 胜负判断

<img src="https://dzy-typora-img-hosting-service.oss-cn-shanghai.aliyuncs.com/typoraImgs/image-20200901180357729.png" alt="image-20200901180357729" style="zoom:50%;" />

<img src="https://dzy-typora-img-hosting-service.oss-cn-shanghai.aliyuncs.com/typoraImgs/image-20200901180923256.png" alt="image-20200901180923256" style="zoom:50%;" />

<img src="https://dzy-typora-img-hosting-service.oss-cn-shanghai.aliyuncs.com/typoraImgs/image-20200901180633605.png" alt="image-20200901180633605" style="zoom:50%;" />

<img src="https://dzy-typora-img-hosting-service.oss-cn-shanghai.aliyuncs.com/typoraImgs/image-20200901180845569.png" alt="image-20200901180845569" style="zoom:50%;" />

<img src="https://dzy-typora-img-hosting-service.oss-cn-shanghai.aliyuncs.com/typoraImgs/image-20200901180755029.png" alt="image-20200901180755029" style="zoom:50%;" />

<img src="https://dzy-typora-img-hosting-service.oss-cn-shanghai.aliyuncs.com/typoraImgs/image-20200901180827240.png" alt="image-20200901180827240" style="zoom:50%;" />

### 3.7 悔棋

<img src="https://dzy-typora-img-hosting-service.oss-cn-shanghai.aliyuncs.com/typoraImgs/image-20200901181018015.png" alt="image-20200901181018015" style="zoom:50%;" />

## 4 核心代码

### 4.1 游戏结束

路径：src/js/AI/analyser.js

判断当前棋子是否导致游戏结束（不包括平局）

```javascript
  static isWin(board, place) {
    if (place === undefined) {
      throw new Error('分析的位置不能为空');
    }

    const {
      color,
    } = place;
    const {
      nWin,
    } = board;
    let winCount;

    if (nWin === 1) {
      return true;
    }
    const analysisArrays = this.getPlaceAnalysisArrays(board, place, nWin);
    for (let direction = 0; direction < analysisArrays.length; direction += 1) {
      winCount = 0;
      for (const value of analysisArrays[direction]) {
        if (value === color) {
          winCount += 1;
          if (winCount >= nWin) {
            return true;
          }
        } else {
          winCount = 0;
        }
      }
    }
    return false;
  }
```

### 4.2 判断禁手

路径：src/js/AI/analyser.js

判断是否有“禁手”限制，禁手指在五子棋比赛中不允许的走法（否则执黑先行有巨大优势）。

```javascript
	static isFoul(board, place) {
    if (place === undefined) {
      throw new Error('分析的位置不能为空');
    }
    const {
      color,
    } = place;
    const {
      nWin,
    } = board;
    const range = nWin + 1;
    if (color === 1) {
      return false;
    }
    let noooooo = 0;
    let noooo = 0;
    let nooo = 0;
    const analysisArrays = this.getPlaceAnalysisArrays(board, place, range);
    for (const analysisArray of analysisArrays) {
      const [oooooo, , oooo, xoooo, ooo] = Analyser.getWinCount(nWin, color, analysisArray);
      noooooo += oooooo;
      noooo += oooo + xoooo;
      nooo += ooo;
    }
    if (noooooo > 0) { // 长连禁手
      return '长连禁手';
    }
    if (nooo === 2) { // 33,433
      if (noooo === 0) {
        return '三三禁手';
      }
      if (noooo === 1) {
        return '四三三禁手';
      }
    } else if ((noooo === 2) && nooo < 2) { // 44,344
      if (nooo === 0) {
        return '四四禁手';
      }
      if (nooo === 1) {
        return '三四四禁手';
      }
    }
    return false;
  }
```

### 4.3 估值函数

路径：src/js/AI/analyser.js

获得单个棋子的分数。

```javascript
 const range = nWin + 1;
    let score = 0;

    let analysisArrays = Analyser.getPlaceAnalysisArrays(board, place, range);
    for (const analysisArray of analysisArrays) {
      const [, ooooo, oooo, xoooo, ooo, xooo, oo,
        xoo,
      ] = Analyser.getWinCount(nWin, place.color, analysisArray);
      if (ooooo > 0) {
        return 100000;
      }
      const [, xxxxx, xxxx, oxxxx, xxx, oxxx, xx,
        oxx,
      ] = Analyser.getWinCount(nWin, -place.color, analysisArray);
      score += (ooooo - xxxxx) * 10000
        + (oooo - xxxx) * 1000 + (xoooo - oxxxx) * 300
        + (ooo - xxx) * 200 + (xooo - oxxx) * 30 + (oo - xx) * 30 + (xoo - oxx) * 1;
    }
    const backup = place.color;
    place.color = 0;
    analysisArrays = Analyser.getPlaceAnalysisArrays(board, place, range);
    for (const analysisArray of analysisArrays) {
      const [, ooooo, oooo, xoooo, ooo, xooo, oo,
        xoo,
      ] = Analyser.getWinCount(nWin, backup, analysisArray);
      const [, xxxxx, xxxx, oxxxx, xxx, oxxx, xx,
        oxx,
      ] = Analyser.getWinCount(nWin, -backup, analysisArray);
      score -= (ooooo - xxxxx) * 10000 + (oooo - xxxx) * 1000
        + (xoooo - oxxxx) * 300 + (ooo - xxx) * 200
        + (xooo - oxxx) * 30 + (oo - xx) * 20 + (xoo - oxx) * 1;
    }
    place.color = backup;
    return score;
  }
  
  static getWinCount(nWin, color, analysisArray) {
    const stackMaxLength = nWin + 1;
    const analysisStack = [];
    let count = 0;
    let oooooo = 0;
    let ooooo = 0;
    let oooo = 0;
    let xoooo = 0;
    let ooo = 0;
    let xooo = 0;
    let oo = 0;
    let xoo = 0;
    let lastColor = -color;
    let endBlock = 0;

    for (let index = 0; index < analysisArray.length; index += 1) {
      let value = isNaN(analysisArray[index]) ? -color : analysisArray[index];
      if (value === color || analysisStack.length > 0) {
        if (analysisStack.length === 0) {
          if (lastColor !== -color) {
            let researchIndex = analysisArray.lastIndexOf(-color, index);
            researchIndex = Math.max(researchIndex, Math.max(index - nWin + 2, 0));
            researchIndex = analysisArray.indexOf(color, researchIndex);
            if (researchIndex < index && researchIndex > 0) {
              index = researchIndex; // >=1
              value = analysisArray[index];
              lastColor = analysisArray[index - 1];
            }
          }

          analysisStack.push(lastColor);
        }

        analysisStack.push(value);

        // 计算连在一起的棋形
        if (analysisStack.length >= stackMaxLength
          || value === -color || index >= analysisArray.length - 1) {
          count = analysisStack.reduce((p, v) => p + (v === color ? 2 : 0), 0);
          if (count >= nWin * 2) {
            ooooo += 1; // 胜利
            if (color === -1 && (count > nWin * 2 || analysisArray[index + 1] === color)) {
              oooooo += 1; // 黑方禁手
            }
          }
          endBlock = analysisStack.lastIndexOf(color) + 1;
          endBlock = endBlock > analysisStack.length - 1 ? analysisStack.length - 1 : endBlock;
          endBlock = analysisStack[endBlock];
          if (endBlock !== 0) { // ooox
            count -= 1;
          } else if (analysisStack[0] !== 0) { // xooo
            count -= 1;
          }
          if (index - analysisArray.lastIndexOf(-color, index - 1)
            <= ((analysisStack[analysisStack.length - 1] === -color ? nWin : nWin - 1))) { // x_oo_x
            count = -10000;
          }
          switch (nWin * 2 - count) {
            case 2:
              oooo += 1;
              break;
            case 3:
              xoooo += 1;
              break;
            case 4:
              ooo += 1;
              break;
            case 5:
              xooo += 1;
              break;
            case 6:
              oo += 1;
              break;
            case 7:
              xoo += 1;
              break;
            default:
              break;
          }
          analysisStack.length = 0;
        }
      }
      lastColor = value;
    }
    return [oooooo, ooooo, oooo, xoooo, ooo, xooo, oo, xoo];
  }
```

### 贪心算法AI

路径：src/js/AI/simpleAI.js

  获取可以下的点的数组，并做了排序，可以单独做为一个五子棋AI来使用，但效果不好。

```javascript
 getNexts(color, searchRange) {
    const openlist = [];
this.board.data.forEach((value, index) => {
  if (value !== 0) {
    return;
  }
  let place;
  const {
    x,
    y,
  } = this.board.getXY(index);
  for (let i = -searchRange; i < searchRange + 1; i += 1) {
    for (let j = -searchRange; j < searchRange + 1; j += 1) {
      if (i === 0 && j === 0) {
        continue;
      }
      const flag = this.board.data[this.board.getIndex(x + i, y + j)];
      if (flag === color || flag === -color) {
        place = {
          x,
          y,
          color,
          score: 0,
        };
        place.score = Analyser.getPlaceScore(this.board, place);
        openlist.push(place);
        return;
      }
    }
  }
});
```
### 4.4 博弈树AI

路径：src/js/AI/gameTreeAI.js

博弈树AI，继承自贪心AI，依赖于分析类

```javascript
 think(color, depth, alpha, beta, foulRule) {
    // 如果是叶子节点则直接返回分数
    if (depth < 1) {
      return Analyser.getScore(this.board, color) - Analyser.getScore(this.board, -color);
    }
    const openlist = this.getNexts(color, depth === this.maxDepth ? this.searchRange : 1);
    if (openlist.length === 0) {
      return 0; // 将平局分数设置为零，不然AI会避免平局
    }
    openlist.length = Math.min(openlist.length, this.maxBreadth); // 启发式减枝，限制每次递归的广度
    // 遍历每一个可行下法
    for (const place of openlist) {
      if (this.best === null) {
        this.best = place; // 默认值为由贪心算法决定的估值最大的一个下法
      }
      // 超时检测
      if (this.maxDepth === depth && this.best !== null) {
        if ((new Date()).getTime() - this.lasttime.getTime() > this.timelimit) {
          this.timeout = true;
          return 0;
        }
      }
      this.board.placeStone(place); // 下棋
      if (Analyser.isWin(this.board, place)) {
        place.score = 100000 + depth;
      } else if (foulRule && Analyser.isFoul(this.board, place)) {
        place.score = -100000 - depth;
      } else {
        place.score = -this.think(-color, depth - 1, -beta, -alpha, foulRule);
      }
      this.board.undo(place); // 还原棋盘

      if (place.score > alpha) {
        alpha = place.score; // 下界提升
        if (depth === this.maxDepth) {
          this.best = place;
        }
      }
      if (place.score >= beta) {
        break; // 减枝
      }
    }
    return alpha;
  }
```

迭代深化：

将来可以进一步完善，将最优点加入置换表 然后在启发式搜索函数中获取将此下法的排序提前。

```javascript
 iterativeDeepening(color, foulRule) {
    this.lasttime = new Date();
    this.timeout = false;
    this.best = null;
    const maxDepthOld = this.maxDepth;

    for (let depth = 1; depth <= maxDepthOld; depth += 1) {
      this.maxDepth = depth;
      this.think(color, depth, -1000000, 1000000, foulRule);
      if (this.timeout || this.best === null) {
        break;
      }
      if (this.best !== null && this.best.score >= 100000) {
        break;
      }
    }
    this.timeout = false;
    if (this.best) {
      this.best.depth = this.maxDepth;
    }
    this.maxDepth = maxDepthOld;
  }
```

### 4.5 判断嵌套

路径：src/js/objEqual.js

判断两个嵌套的object是否相等。

```javascript
class Equal {
  static equal(any1, any2) {
    const type1 = typeof (any1);
    const type2 = typeof (any2);
    if (type1 !== type2) {
      return false;
    }
    if (type1 === 'function') {
      if (any1.toString() === any2.toString()) {
        return true;
      }
    } else if (type1 === 'object') {
      if (any1 === any2) {
        return true;
      }
      if (any1 === null || any2 === null) {
        return false;
      }
      if (any1.toString() !== any2.toString()) {
        return false;
      }
      for (const k in any1) {
        if (any1.hasOwnProperty(k)) {
          if (any2.hasOwnProperty(k)) {
            if (!Equal.equal(any2[k], any1[k])) {
              return false;
            }
          } else {
            return false;
          }
        }
      }
      return true;
    }
    return any1 === any2;
  }
}
```

### 4.6 绘制渲染类

路径：src/js/renderer.js

绘制落子提示。

```JavaScript
  drawPointer(offsetX, offsetY, gridSize, color) {
    if (isNaN(offsetX) || isNaN(offsetY)) {
      return;
    }
    const x = (Math.round(offsetX / gridSize - 0.5) + 0.5) * gridSize;
    const y = (Math.round(offsetY / gridSize - 0.5) + 0.5) * gridSize;
    this.context.beginPath();
    this.context.arc(x, y, gridSize / 2 - 1, 0, 360, false);
    if (color === EStoneColor.black) {
      this.context.fillStyle = '#000000AA';
      this.context.fill();
    } else if (color === EStoneColor.white) {
      this.context.fillStyle = '#FFFFFFAA';
      this.context.fill();
    }
    this.context.closePath();
  }
```

