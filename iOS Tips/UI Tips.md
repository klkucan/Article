- Tom猫的实现是在imageView中播放一组图片动画。
- 大量图片生成时不要使用`UIImage *image = [UIImage imageNamed:@"imageName"];`因为这个方法会把图片加入到Cache中，过多时会内存不足。需要使用

```
NSString *path = [[NSBuddle mainBuddle] pathForResource:fileName oftype@"png"];
UIImage *image = [[UIImage imageWithContentsOfFile:path];
```
这个方法产生的图片不会加入Cache。

- 图片加入Cache是不是完全不好呢？也不是这样，当你需要在一个UI上显示多次同一张图片时，使用Cache是能够节约资源的。因为多个图片引用了同一个Cache中的图片。