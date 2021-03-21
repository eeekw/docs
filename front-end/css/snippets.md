### 单行省略

```css
p {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
```

### 多行省略

```css
p {
  overflow : hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}
```