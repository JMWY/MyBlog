# Linux 上设置垃圾箱

## /usr/bin 下添加 rm 脚本
```sh
#!/bin/sh
##init
source /etc/profile
source ~/.bash_profile
Date=$(date +%F)
Ldate=$(date -d "-3 day" +%Y%m%d)
Fdate=$(date +%F' '%T)
if ! cat ~/.bash_profile|grep -qs 'myrm.sh' -a  [ -s /usr/bin/myrm.sh ] ;then
    echo '
    alias rm="/usr/bin/myrm.sh"
    '>>~/.bash_profile
fi
if ! grep  -qs '.recycle' ~/.bash_profile ;then
    echo "
    ls -ld   \${HOME}/.recycle/* 2>/dev/null |grep \"^d\"|awk '{print \$NF}'|head -n-3|xargs /bin/rm -rf  &>/dev/null
    ">> ~/.bash_profile
fi
if [ "x${LOGNAME}" == "xroot" ];then
    if ! grep  -qs '.recycle' /etc/profile ;then
        echo "
        ls -ld   \${HOME}/.recycle/* 2>/dev/null|grep \"^d\"|awk '{print \$NF}'|head -n-3|xargs /bin/rm -rf &>/dev/null
        ">> /etc/profile
    fi
fi
for item in $*
do
    if [ "x$item" == "x/"  ] ;then
        echo "Error: no permit ,please connect administrator"
        exit 2
    fi
done
dir="${HOME}/.recycle"
[ ! -d ${dir}/${Date} ] && mkdir -p ${dir}/${Date}
echo "$Fdate ${LOGNAME} cmd: rm $*">>${dir}/${Date}/rmcmd.log
plist=$(echo $*)
num=$#
last=$(eval echo \$$num)
for item in $*
do
    if [ "x$item" == "x-h" -o "x$item" == "x--help" ] ;then
        /bin/rm --help
        exit
    fi
    if [ "x$item" == "x-r" -o "x$item" == "x-f" -o "x$item" == "x-rf" -o "x$item" == "x-fr" -o "x$item" == "x-i"  ] ;then
        continue
    else
        
        if  [ -d $item ] ;then
            [ $1 ] && {
            if [ "x$1" == "x-r" -o "x$1" == "x-rf" -o "x$1" == "x-fr"  -o "x$last" == "x-r" -o "x$last" == "x-rf" -o "x$last" == "x-fr" ] ;then
                :
            elif  echo $plist|grep -qs '\-r' 2>/dev/null;then
                :
            else
                echo "Error: $item is a directory"
                exit 2
            fi    
        } || { 
        echo  "Error: no file or directory to rm "
    }
        fi
        if [ -e ${dir}/${Date}/$item ];then
            mv  $item ${dir}/${Date}/${item}.$$
        else
            mv  $item ${dir}/${Date}/
        fi
    fi
done
##clear
tmp=
for tmp in `ls -ld   ${HOME}/.recycle/* 2>/dev/null |grep "^d" |grep -oP "\d{4}-\d{2}-\d{2}"`
do 
    tmp_date=$(date -d "$tmp" +%Y%m%d)
    if [ $tmp_date -lt $Ldate ];then
        /bin/rm -rf ${HOME}/.recycle/$tmp_date
    fi
done

```

## 添加 rm 别名
alias rm='/usr/bin/myrm.sh'

## 添加删除的脚本

```sh
# save 3 days
ls -ld   ${HOME}/.recycle/* 2>/dev/null |grep "^d"|awk '{print $NF}'|head -n-3|xargs /bin/rm -rf  &>/dev/null
```
