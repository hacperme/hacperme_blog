# hacperme_blog

My  personal website.



## 使用方法

1. 本地渲染

   ```bash
   hugo server --disableFastRender -D
   ```

   Web Server is available at http://localhost:1313/

2. 添加新文章

   ```bash
   hugo new posts/draft/xxx.md
   ```

3. 添加图集

   使用 shortcode 的方式

   ```bash
   # 启用 lightbox，添加下面的 {{< load-photoswipe >}} shortcode 
   {{< load-photoswipe >}}
   
   # 单张图片 {{< figure >}}  shortcode 
   {{< figure link="image.jpg" >}}
   # image.jpg 要放在 static/images 路径下
   # link 参数也可以填图片的 url
   
   # 多张图片 {{< gallery >}} shortcode 
   {{< gallery dir="/img/your-directory-of-images/" />}}
   # 从指定文件夹加载图片， 图片要放在static/images 路径下
   
   # 指定特定图片
   {{< gallery >}}
       {{< figure link="/img/homepage/sydney-harbour.jpg" caption="Sydney Harbour" >}}
       {{< figure link="/img/homepage/cc_jeepers.jpg" caption="Capital Chorus" >}}
       {{< figure link="/img/arduino/test-setup.jpg" caption="Arduino test setup" >}}
   {{< /gallery >}}
   
   #  {{< figure >}} {{< gallery >}} 可设定的参数
   caption
   # 自定义标题
   {{< figure src="/img/homepage/sydney-harbour.jpg"
   	width="400px" caption="Sydney Harbour" >}}
   caption-position
   # 标题位置
    -bottom (default)
    -center
    -none 
   {{< gallery hover-effect="none" caption-effect="slide" >}} ...
   {{< gallery hover-effect="none" caption-effect="fade" >}} ...
   caption-effect
   # 标题特效
    -slide (default)
    -fade
    -none (captions always visible)
    {{< gallery caption-position="bottom" caption-effect="slide" >}} ...
   {{< gallery caption-position="center" caption-effect="fade" >}} ...
   hover-effect
   # 选中图片特效
   -zoom (default)
   -grow
   -shrink
   -slideup
   -slidedown
   -none
   {{< gallery hover-effect="grow" >}} ...
   {{< gallery hover-effect="shrink" >}} ...
   ```

4. 引用 bilibili 视频

   ```bash
   {{< bilibili bvid >}}   
   {{< bilibili bvid 5 >}}
   ```

   



