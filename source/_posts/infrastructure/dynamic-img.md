---
title: 前端动态图片管理方案
date: 2021-02-25 18:02:30
categories:
  - web
  - infrastructure
tags:
  - web
  - infrastructure
---

为什么要采取动态图片管理方案

## 优势

- 数据表存取的是图片名，存取图片不需要考虑路径问题
- 图片路径即使修改，也无需改代码
- 组件显示图片更方便
- 图片可以分类管理

<!-- more -->

## 安装

```sh
npm install good-storage -S
```

## 怎么做

通过import.meta.glob读取所有图片，生成一个对象，key是图像名称，value是绝对路径

定义一个ImgUtil类，可以使用静态方法，无需专门new ImgUtil()

```javascript
import goodStorage from 'good-storage';
import { isEmpty as isObjectEmpty } from '@acemall/mall-utils';
export class ImgUtil {

  static imgList: Record<string, string> = {};
  /**
   * Use import.meta.glob to read all images, generate an object, 
   * key is image's name, value is image absolute path.
   * @returns { imgname: absolute path, imgname: absolute path, ...}
   */
  static loadAllImgs() {
    const imgsMap = import.meta.glob('../assets/img/**/*.png', {
      eager: true
    })
    let abPath: string = ''; // absolute path
    let imgName: string = ''
    for(const key in imgsMap) {
      abPath = (imgsMap[key] as any).default;
      imgName = abPath.substring(abPath.lastIndexOf('/') + 1);
      this.imgList[imgName] = abPath;
    }
  }

  /**
   * get image path from image name
   * @param imgName 
   * @returns 
   */
  static getImg(imgName: string) {
    return ImgUtil.imgList[imgName];
  }

  /**
   * trigger to cached image absolute path to local storage
   */
  static storageImgList() {
    this.imgList = goodStorage.get('imgList') || {}
    if(this.isEmpty()) {
      this.loadAllImgs();
      goodStorage.set('imgList', this.imgList);
    }
  }

  /**
   * 
   * @returns object is empty
   */
  static isEmpty() {
    return isObjectEmpty(this.imgList);
  }
}
```

缓存路径成功

![](/pics/infrastructure/img-storage.png)

使用

```javascript
import { ImgUtil } from 'path to ImgUtil or package'
// 初始化的时候执行这个静态方法，就会触发读取本地图片，缓存到local storage
ImgUtil.storageImgList()
```

```html
<img :src="ImgUtil.get('xxx.png')" />
```



