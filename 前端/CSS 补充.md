# CSS

## 1. 标签选择器

例子 http://www.mamicode.com/info-detail-2300094.html

### 1.1 坑

#### 1.1.1 空格

```CSS
div[class="test"]{
    color: red;
    background-color: #1E9FFF;
}
```

```css
div [class="test"]{
    color: red;
    background-color: #1E9FFF;
}
```

* 第一个是正确的，第二个是错误的 空格问题

#### 1.1.2 数字

id不能数字开头，否则无法识别

