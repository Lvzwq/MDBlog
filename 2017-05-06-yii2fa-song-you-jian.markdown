---
layout: post
title: "YII2发送邮件"
date: 2015-11-22 23:06:47 +0800
comments: true
categories: Dev
tags: [PHP]
---

<!--more-->
`YII2`是`PHP`中一个比较流行的WEB框架。
下面来讲一下在`YII2`中如何配置邮件服务。`YII2`中默认是使用`swiftmailer`来发送邮件的。
如果`YII2`中没有这个扩展，可以使用以下方式安装
```sh
php composer.phar require --prefer-dist yiisoft/yii2-swiftmailer
```
或者添加以下代码到`composer.json`
```
"yiisoft/yii2-swiftmailer": "~2.0.0"
```
然后执行`composer install`命令更新扩展

安装完之后，需要在`YII2`的配置文件中配置邮箱的基本信息
在`config/web.php`中添加
```Php
'mailer' => [
			'class' => 'yii\swiftmailer\Mailer',
			'viewPath' => '@app/mail',  //配置邮件模板的位置
			'transport' => [
				'class' => 'Swift_SmtpTransport',
				'host' => 'smtp.126.com',
				'username' => 'xxuser@126.com',
				'password' => 'xxpwd',
				'port' => '465',
				'encryption' => 'ssl',
			],
			'useFileTransport' => false,
            'messageConfig'=> [
                'charset'=>'UTF-8',
                'from'=>['xxuser@126.com'=>'admin']
            ],
		],
```
## 简单的邮件
```
Yii::$app->mailer->compose()
     ->setFrom('xxuser@126.com')
     ->setTo($mailTo)
     ->setTextBody('您好， 有邮件来访')   //发布纯文字文本
     //->setHtmlBody("点击查看<a href='zwq.bingyan.net'>zwq.bingyan.net</a>")    //发布可以带html标签的文本
     ->setSubject($subject)
     ->send();
```

## 带有模板的邮件
```
 $mail = Yii::$app->mailer->compose('resetPassword', [ "id" => 123, "key" => "asdsfdgfrtryt124354"]);
        $mail->setFrom(Yii::$app->params['adminEmail']);
        $mail->setTo("xxuser@gmail.com");
        $mail->setSubject("XX网 | 密码重置");
        return $mail->send();
```
上面的代码说明，使用`resetPassword.php`这个模板,模板文件在刚才配置的`viewPath`这个目录下。
另外这个模板默认是以一个公共的模板`layouts/html.php`配置的。发送带有模板的邮件是不需要调用`setTextBody`和`setHtmlBody`的。
模板文件目录结构如下所示
```
-- mail/
    -- layouts/
        -- html.php
    -- resetPassword.php
```
其中`html.php`
```php
<?php
use yii\helpers\Html;

/* @var $this \yii\web\View view component instance */
/* @var $message \yii\mail\MessageInterface the message being composed */
/* @var $content string main view render result */
$resetURL = Yii::$app->urlManager->createAbsoluteUrl(['user/activate']);
?>
<?php $this->beginPage() ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=<?= Yii::$app->charset ?>" />
    <title><?= Html::encode($this->title) ?></title>
    <?php $this->head() ?>
</head>
<body>
    <?php $this->beginBody() ?>
    <?= $content ?>
    <?php $this->endBody() ?>
</body>
</html>
<?php $this->endPage() ?>
```
`resetPassword.php`中
```
<?php
use yii\helpers\Html;

$resetLink = Yii::$app->urlManager->createAbsoluteUrl(['user/activate', 'id' => $id, "key" => $key]);
?>

<p>尊敬的用户</p>
<p>您好!</p>
<b>请点击以下链接重置密码</b><br>
<a href="<?= $resetLink ?>"><?= $resetLink ?></a>
<br><br>
<p>系统发信，请勿回复</p>
```
在`resetPassword.php`最后解析的`html`内容会替换到`html.php`中的`$content`.
在向模板中传递参数时，是以数组的形式传递的，如上`$id`, `$key`和`compose`中第二个参数保持一致。

根据上面配置会生成
`http://www.example.com/user/activite?id=123&key=asdsfdgfrtryt124354`
这个链接不够友好，我们可以使Ta更加`Restful`
`http://www.example.com/user/activite/123/asdsfdgfrtryt124354`
这样需要在`config/web.php`中配置
```
'urlManager' => [
			'showScriptName' => false,
			'enablePrettyUrl' => true,
			//'suffix' => '.htm',
			'rules' => [
                "<controller:(user)>/<action:(activate)>/<id:\d+>/<key:\w+>" => '<controller>/<action>',
			],
		],
```

更详细，请参考[官方教程](http://www.yiiframework.com/doc-2.0/ext-swiftmailer-index.html)


