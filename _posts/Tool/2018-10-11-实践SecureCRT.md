---
layout: post
title: "实践SecureCRT"
date: 2018-10-11 09:58:13 +0800
categories: 工具
tags: SecureCRT tool SecureFX
---

## 安装

deepin15.7下安装

```shell
$ apt list scrt-sfx
正在列表... 完成
scrt-sfx/panda 8.0.2-1118 amd64
$ sudo apt install scrt-sfx
```

破解

```shell
$ wget http://download.boll.me/securecrt_linux_crack.pl
$ sudo perl securecrt_linux_crack.pl /usr/bin/SecureCRT 
crack successful

License:

	Name:		xiaobo_l
	Company:	www.boll.me
	Serial Number:	03-94-294583
	License Key:	ABJ11G 85V1F9 NENFBK RBWB5W ABH23Q 8XBZAC 324TJJ KXRE5D
	Issue Date:	04-20-2017
```

运行SecureCRT，会提示注册，选择“Enter License Manually”，然后输入上述信息即可。

参考[Linux MacOSX SecureCRT 完全破解](https://www.boll.me/archives/680#comment-2421)

参照securecrt_linux_crack.pl，编写了securefx_linux_crack.pl，用于破解SecureFX

```shell
$ sudo perl securefx_linux_crack.pl /usr/bin/SecureFX
crack successful

License:

	Name: ygeR
	Company: TEAM ZWT
	Serial Number:06-70-001589
	License Key: ACUYJV Q1V2QU 1YWRCN NBYCYK ABU767 D4PQHA S1C4NQ GVZDQF
	Issue Date: 03-10-2017
```

## 超时

可以通过两个入口进行设置： 
1、右击Session中的连接，选择Properties->Terminal->Anti-idle->勾选Send protocol NO-OP。 
2、当已经建立连接的情况，可以通过上面方法，另外也可以通过点击Options->Session Options->Terminal->Anti-idle->勾选Send protocol NO-OP

选中之后，后面显示60秒是默认值，可对其进行增减，只要这个时间段小于自动断开连接的时间就可以了。



## securecrt_linux_crack.pl

```perl
#!/usr/bin/perl 
#===============================================================================
#
#         FILE: securecrt_linux_crack.pl
#
#        USAGE: ./securecrt_linux_crack.pl  
#
#  DESCRIPTION: securecrt_linux_crack
#
#      OPTIONS: ---
# REQUIREMENTS: ---
#         BUGS: ---
#        NOTES: ---
#       AUTHOR: xiaobo_l
# ORGANIZATION: 
#      VERSION: 1.2
#      CREATED: 08/16/2015 13:26:00 
#     REVISION: ---
#===============================================================================

use strict;
use warnings;
use File::Copy qw(move);




sub license {
	print "\n".
	"License:\n\n".
	"\tName:\t\txiaobo_l\n".
	"\tCompany:\twww.boll.me\n".
	"\tSerial Number:\t03-94-294583\n".
	"\tLicense Key:\tABJ11G 85V1F9 NENFBK RBWB5W ABH23Q 8XBZAC 324TJJ KXRE5D\n".
	"\tIssue Date:\t04-20-2017\n\n\n";
}

sub usage {
    print "\n".
	"help:\n\n".
	"\tperl securecrt_linux_crack.pl <file>\n\n\n".
	"\tperl securecrt_linux_crack.pl /usr/bin/SecureCRT\n\n\n".
    "\n";
	
	&license;

    exit;
}
&usage() if ! defined $ARGV[0] ;


my $file = $ARGV[0];

open FP, $file or die "can not open file $!";
binmode FP;

open TMPFP, '>', '/tmp/.securecrt.tmp' or die "can not open file $!";

my $buffer;
my $unpack_data;
my $crack = 0;

while(read(FP, $buffer, 1024)) {
	$unpack_data = unpack('H*', $buffer);
	if ($unpack_data =~ m/785782391ad0b9169f17415dd35f002790175204e3aa65ea10cff20818/) {
		$crack = 1;
		last;
	}
	if ($unpack_data =~ s/6e533e406a45f0b6372f3ea10717000c7120127cd915cef8ed1a3f2c5b/785782391ad0b9169f17415dd35f002790175204e3aa65ea10cff20818/ ){
		$buffer = pack('H*', $unpack_data);
		$crack = 2;
	}
	syswrite(TMPFP, $buffer, length($buffer));
}

close(FP);
close(TMPFP);

if ($crack == 1) {
		unlink '/tmp/.securecrt.tmp' or die "can not delete files $!";
		print "It has been cracked\n";
		&license;
		exit 1;
} elsif ($crack == 2) {
		move '/tmp/.securecrt.tmp', $file or die 'Insufficient privileges, please switch the root account.';
		chmod 0755, $file or die 'Insufficient privileges, please switch the root account.';
		print "crack successful\n";
		&license;
} else {
	die 'error';
}
```

## securefx_linux_crack.pl

```perl
#!/usr/bin/perl 
#===============================================================================
#
#         FILE: securefx_linux_crack.pl
#
#        USAGE: ./securefx_linux_crack.pl  
#
#  DESCRIPTION: securefx_linux_crack
#
#      OPTIONS: ---
# REQUIREMENTS: ---
#         BUGS: ---
#        NOTES: ---
#       AUTHOR: xiaobo_l
# ORGANIZATION: 
#      VERSION: 1.2
#      CREATED: 08/16/2015 13:26:00 
#     REVISION: ---
#===============================================================================

use strict;
use warnings;
use File::Copy qw(move);




sub license {
	print "\n".
	"License:\n\n".
	"\tName:\t\tygeR\n".
	"\tCompany:\tTEAM ZWT\n".
	"\tSerial Number:\t06-70-001589\n".
	"\tLicense Key:\tACUYJV Q1V2QU 1YWRCN NBYCYK ABU767 D4PQHA S1C4NQ GVZDQF\n".
	"\tIssue Date:\t03-10-2017\n\n\n";
}

sub usage {
    print "\n".
	"help:\n\n".
	"\tperl securefx_linux_crack.pl <file>\n\n\n".
	"\tperl securefx_linux_crack.pl /usr/bin/SecureFX\n\n\n".
    "\n";
	
	&license;

    exit;
}
&usage() if ! defined $ARGV[0] ;


my $file = $ARGV[0];

open FP, $file or die "can not open file $!";
binmode FP;

open TMPFP, '>', '/tmp/.securefx.tmp' or die "can not open file $!";

my $buffer;
my $unpack_data;
my $crack = 0;

while(read(FP, $buffer, 1024)) {
	$unpack_data = unpack('H*', $buffer);
	if ($unpack_data =~ m/e02954a71cca592c855c91ecd4170001d6c606d38319cbb0deabebb05126/) {
		$crack = 1;
		last;
	}
	if ($unpack_data =~ s/c847abca184a6c5dfa47dc8efcd700019dc9df3743c640f50be307334fea/e02954a71cca592c855c91ecd4170001d6c606d38319cbb0deabebb05126/ ){
		$buffer = pack('H*', $unpack_data);
		$crack = 2;
	}
	syswrite(TMPFP, $buffer, length($buffer));
}

close(FP);
close(TMPFP);

if ($crack == 1) {
		unlink '/tmp/.securefx.tmp' or die "can not delete files $!";
		print "It has been cracked\n";
		&license;
		exit 1;
} elsif ($crack == 2) {
		move '/tmp/.securefx.tmp', $file or die 'Insufficient privileges, please switch the root account.';
		chmod 0755, $file or die 'Insufficient privileges, please switch the root account.';
		print "crack successful\n";
		&license;
} else {
	die 'error';
}
```

