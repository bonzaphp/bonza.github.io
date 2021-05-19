---
layout: post
title:  "thinkphp响应gd库生成的图片"
categories: thinkphp
tags: thinkphp extension-gd
author: HyG
---

* content
{:toc}

1. 先检查是否有gd库
2. 绘制相应的点


```php
#检查是否存在必要扩展
if (extension_loaded('gd')) {
            //放大倍数
            $pix = 4;
            //加粗倍数
            $bold = 2;
            // 1.创建画布
            $im = imagecreatetruecolor(140 * $pix, 210 * $pix);
            //2. 修改画布默认填充色为白色
            $white = imagecolorallocate($im, 255, 255, 255);
            imagefill($im, 0, 0, $white);
            //创建一个颜色
            $black = imagecolorallocate($im, 0, 0, 0);
            $red = imagecolorallocate($im, 255, 0, 0);
            //矩形
//            imagerectangle($im,2,2,40,50,$red);
            //从数据库拿数据
            $data = $this->getDrawData();
//            dump($data);
            //            foreach ()
            //画点
            foreach ($data as $v) {
                //绘制一个点
                imagesetpixel($im, $v['x'] * $pix, $v['y'] * $pix, $black);
                //绘制一个圆，相当于加粗之前的点
                imageellipse($im, $v['x'] * $pix, $v['y'] * $pix, $bold, $bold, $black);
            }
            //3.输出图像到网页，也可以另存
            ob_start();
            imagepng($im);
            $content = ob_get_clean();
            //4.销毁该图片（释放内存--服务器内存）
            imagedestroy($im);
            return response($content, 200, ['Content-Length' => strlen($content)])->contentType('image/png');
        }
```
