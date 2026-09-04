# 华为手表端检查清单

## 文件说明

| 文件 | 用途 | 大小 |
|------|------|------|
| `index.html` | 手机端/网页端 | 23,040 字节 |
| `watch.html` | 手表端专用 | 18,392 字节 |

---

## 手表端适配检查项

### 1. 屏幕适配

- [x] **454×454 圆形手表** (GT系列)
  - CSS: `@media(min-width:400px)` 优化网格布局
  - 迷宫最大显示区域

- [x] **194×368 长条手表** (Fit系列)
  - CSS: `@media(max-width:250px)` 限制画布大小
  - 单列按钮布局

- [x] **迷宫尺寸限制**
  - 手机端：最大 75×75
  - **手表端：最大 31×31** ✅

### 2. 存储适配

| 环境 | API | 状态 |
|------|-----|------|
| 手机/网页 | `localStorage` | ✅ 保留 |
| **华为手表** | **`@system.storage`** | ✅ 已适配 |

**关键代码：**
```javascript
const Storage = {
  isWatch: typeof require !== 'undefined',
  
  get: function(key, callback) {
    if (this.isWatch) {
      const storage = require('@system.storage');
      storage.get({key: key, success: (data) => callback(data.value)});
    } else {
      callback(localStorage.getItem(key));
    }
  },
  
  set: function(key, value) {
    if (this.isWatch) {
      const storage = require('@system.storage');
      storage.set({key: key, value: value});
    } else {
      localStorage.setItem(key, value);
    }
  }
};
```

### 3. 输入控制

| 输入方式 | 手机端 | 手表端 | 状态 |
|----------|--------|--------|------|
| 键盘 | ✅ WASD/方向键 | ❌ 无键盘 | - |
| 触摸按钮 | ✅ 方向键 | ❌ 屏幕小 | - |
| **触摸滑动** | ✅ | ✅ | ✅ 保留 |
| **表冠旋转** | ❌ | ✅ | ✅ 新增 |
| 点击确认 | ❌ | ✅ | ✅ 新增 |

**表冠控制代码：**
```javascript
let crownAccum = 0;
function handleCrown(event) {
  if (event.rotationRate) {
    crownAccum += event.rotationRate.beta || 0;
    if (Math.abs(crownAccum) > 30) {
      if (crownAccum > 0) move(0, -1); // 上
      else move(0, 1);  // 下
      crownAccum = 0;
    }
  }
}

// 华为手表特有API
if (window.device && window.device.onCrown) {
  window.device.onCrown(handleCrown);
}
```

### 4. 功能差异

| 功能 | 手机端 | 手表端 | 说明 |
|------|--------|--------|------|
| 经典关卡 | 30关 (15-75) | 30关 (15-31) | 尺寸限制 |
| 迷雾关卡 | 15关 (17-55) | 15关 (17-31) | 尺寸限制 |
| 迷雾视野 | 4格 | **3格** | 屏幕更小 |
| 钥匙/门 | ✅ | ❌ | 简化 |
| 自定义迷宫 | 5-99 | **5-31** | 尺寸限制 |
| 帮助页面 | ✅ | ✅ | 简化版 |
| 全屏按钮 | ✅ | ❌ | 不需要 |

### 5. 视觉优化

- [x] 字体缩小（适合小屏幕）
- [x] 按钮间距压缩
- [x] 画布自适应
- [x] 表冠操作提示文字

### 6. 待验证项

**必须在真机上测试：**

- [ ] 表冠旋转灵敏度（30度阈值是否合适）
- [ ] 触摸滑动识别（20px阈值是否合适）
- [ ] 存储读写是否正常
- [ ] 454×454 圆形屏幕显示
- [ ] 194×368 长条屏幕显示
- [ ] 电池消耗情况
- [ ] 后台暂停/恢复

---

## 打包要求

### Lite Wearable (GT/Fit系列)

```
文件结构：
├── src
│   └── main
│       └── js
│           └── default
│               └── pages
│                   └── index
│                       ├── index.hml
│                       ├── index.css
│                       └── index.js
│       └── resources
│           └── base
│               └── media
│                   └── icon.png
├── config.json
└── watch.html (复制到 pages/index/)
```

### config.json 关键配置

```json
{
  "app": {
    "bundleName": "com.muliuxing.mazegame",
    "vendor": "Mu_Liuqing",
    "version": {
      "code": 1,
      "name": "1.0.0"
    },
    "apiVersion": {
      "compatible": 4,
      "target": 5,
      "releaseType": "Release"
    }
  },
  "deviceConfig": {
    "default": {
      "network": {
        "cleartextTraffic": true
      }
    }
  },
  "module": {
    "package": "com.muliuxing.mazegame",
    "name": ".MyApplication",
    "mainAbility": ".MainAbility",
    "deviceType": [
      "liteWearable"
    ],
    "distro": {
      "deliveryWithInstall": true,
      "moduleName": "entry",
      "moduleType": "entry"
    },
    "abilities": [
      {
        "name": ".MainAbility",
        "icon": "$media:icon",
        "label": "MazeGame",
        "type": "page",
        "launchType": "standard"
      }
    ]
  }
}
```

---

## 发布前检查

1. [ ] 包大小 < 20MB
2. [ ] 无 `localStorage` 调用
3. [ ] 表冠事件正常
4. [ ] 存储读写正常
5. [ ] 屏幕适配正常
6. [ ] 中文显示正常
7. [ ] 免费关卡 (3-1, 9-1, F-3) 可玩
8. [ ] 付费解锁逻辑正确

---

## 常见问题

### Q: 为什么手表端迷宫更小？
**A:** 手表屏幕小（最大454px），75×75的迷宫每个格子只有6px，无法看清。31×31每个格子约14px，刚好可玩。

### Q: Preferences API 和 localStorage 有什么区别？
**A:** 
- `localStorage`：浏览器标准API，同步读写
- `Preferences`：华为手表专用API，异步读写，需要回调函数

### Q: 表冠旋转不灵敏怎么办？
**A:** 调整 `crownAccum` 阈值（当前30度），或改为检测单次旋转事件而非累积。

### Q: 手表端能独立购买吗？
**A:** 不能。Lite Wearable 不支持应用内购买。必须在手机端购买后同步进度。
