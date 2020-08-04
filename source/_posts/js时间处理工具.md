---
title: js时间处理工具
top: false
cover: false
toc: true
mathjax: true
tags:
  - Date
categories:
  - Web
originContent: ''
date: 2020-08-02 12:56:48
password:
summary:
---


```js
/**
 * 日期格式化
 * yyyy/yy 年
 * MM/M 月
 * E 星期
 * dd/d 日
 * HH/H 小时
 * mm/m 分钟
 * ss/s 秒
 * SS/S 毫秒
 * 除年份、星期 外的配置项 单双位表示，小于10的值是否要前面补0，双位补0，单位不补
 * @param formatStr 格式化字符串 默认为yyyy-MM-dd
 * @returns {*}
 */
Date.prototype.Format = function (formatStr) {
    if(formatStr==''||(typeof formatStr=='undefined'))formatStr='yyyy-MM-dd';
    var Week = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六'];
    formatStr = formatStr.replace(/yyyy/, this.getFullYear());
    formatStr = formatStr.replace(/yy/, (this.getYear() % 100) > 9 ? (this.getYear() % 100).toString() : '0' + (this.getYear() % 100));
    formatStr = formatStr.replace(/MM/, (this.getMonth()+1) > 9 ? (this.getMonth()+1).toString() : '0' + (this.getMonth()+1));
    formatStr = formatStr.replace(/M/g, (this.getMonth()+1));
    formatStr = formatStr.replace(/E/g, Week[this.getDay()]);
    formatStr = formatStr.replace(/dd/, this.getDate() > 9 ? this.getDate().toString() : '0' + this.getDate());
    formatStr = formatStr.replace(/d/g, this.getDate());
    formatStr = formatStr.replace(/HH/, this.getHours() > 9 ? this.getHours().toString() : '0' + this.getHours());
    formatStr = formatStr.replace(/H/g, this.getHours());
    formatStr = formatStr.replace(/mm/, this.getMinutes() > 9 ? this.getMinutes().toString() : '0' + this.getMinutes());
    formatStr = formatStr.replace(/m/g, this.getMinutes());
    formatStr = formatStr.replace(/ss/, this.getSeconds() > 9 ? this.getSeconds().toString() : '0' + this.getSeconds());
    formatStr = formatStr.replace(/s/g, this.getSeconds());
    formatStr = formatStr.replace(/SS/, this.getMilliseconds() > 9 ? this.getMilliseconds().toString() : '0' + this.getMilliseconds());
    formatStr = formatStr.replace(/S/g, this.getMilliseconds());
    return formatStr;
}

/**
 * 获得某月的天数
 * @param paraYear 年
 * @param paraMonth 月
 * @returns {number}
 */
function getMonthDays(paraYear, paraMonth) {
    var monthStartDate = new Date(paraYear, paraMonth - 1, 1);
    var monthEndDate = new Date(paraYear, paraMonth, 1);
    var days = (monthEndDate - monthStartDate) / 86400000;//24 * 60 * 60 * 1000
    return days;
}

/**
 * 获得某月的结束日期
 * @param paraYear 年
 * @param paraMonth 月
 * @returns {string} yyyy-MM-dd
 */
function getMonthEndDate(paraYear, paraMonth) {
    var monthEndDate = new Date(paraYear, paraMonth - 1, getMonthDays(paraYear, paraMonth));
    return monthEndDate.Format('yyyy-MM-dd');
}

/**
 * 获取某年某周的开始日期
 * @param paraYear 年
 * @param weekIndex 周数
 * @returns yyyy-MM-dd
 */
function getBeginDateOfWeek(paraYear, weekIndex) {
    var firstDay = getFirstWeekBegDay(paraYear);
    //7*24*3600000 是一星期的时间毫秒数,(JS中的日期精确到毫秒)
    var time = (weekIndex - 1) * 604800000;//7 * 24 * 3600000
    var beginDay = firstDay;
    //为日期对象 date 重新设置成时间 time
    beginDay.setTime(firstDay.valueOf() + time);
    return beginDay.Format('yyyy-MM-dd');
}


/**
 * 获取某年某周的结束日期
 * @param paraYear 年
 * @param weekIndex 周数
 * @returns yyyy-MM-dd
 */
function getEndDateOfWeek(paraYear, weekIndex) {
    var firstDay = getFirstWeekBegDay(paraYear);
    //7*24*3600000 是一星期的时间毫秒数,(JS中的日期精确到毫秒)
    var time = (weekIndex - 1) * 604800000;//7 * 24 * 3600000
    var weekTime = 518400000;//6天的毫秒数 6 * 24 * 3600000
    var endDay = firstDay;
    //为日期对象 date 重新设置成时间 time
    endDay.setTime(firstDay.valueOf() + weekTime + time);
    return endDay.Format('yyyy-MM-dd');
}

/**
 * 获取某年的第一天
 * @param {*} year 年份
 * @returns Date对象
 */
function getFirstWeekBegDay(year) {
    var tempdate = new Date(year, 0, 1);
    var temp = tempdate.getDay();
    if (temp == 1) {
        return tempdate;
    }
    temp = temp == 0 ? 7 : temp;
    tempdate = tempdate.setDate(tempdate.getDate() + (8 - temp));
    return new Date(tempdate);
}
/**
 * 获取日期为某年的第几周
 * @param date date对象
 * @returns {number} 年周位数
 */
function getWeekIndex(date) {
    var firstDay = getFirstWeekBegDay(date.getFullYear());
    if (date < firstDay) {
        firstDay = getFirstWeekBegDay(date.getFullYear() - 1);
    }
    var d = Math.floor((date.valueOf() - firstDay.valueOf()) / 86400000);
    return Math.floor(d / 7) + 1;
}

/**
 * 获取2个日期之间的日期
 * @param begin 开始 yyyy-MM-dd
 * @param end 结束 yyyy-MM-dd
 * @returns {Array} yyyy-MM-dd日期数组
 */
function getDayBetween(begin, end) {
    var ab = begin.split("-");
    var ae = end.split("-");
    var db = new Date(ab[0], Number(ab[1]), Number(ab[2])).getTime();
    var de = new Date(ae[0], Number(ae[1]), Number(ae[2])).getTime();
    var result = [];
    for (var k = db; k <= de;) {
        result.push(new Date(k).Format('yyyy-MM-dd'));
        k += 86400000;//24 * 60 * 60 * 1000
    }
    return result;
}
/**
 * 数字转中文，0不做处理
 * 数字字符转中文，而不是数字转换中文数字
 * 2018 转换为 二0一八
 * @param num
 * @returns {string} 如：2018 转换为 二0一八
 */
function numberCharToChinese(num) {
    var china = new Array('0', '一', '二', '三', '四', '五', '六', '七', '八', '九');
    var arr = "";
    var english = (Number(num) + "").split("");
    for (var i = 0; i < english.length; i++) {
        arr += china[english[i]];
    }
    return arr;
}
/**
 * 数字转中文
 * @number {Integer} 形如123的数字
 * @return {String} 返回转换成的形如 一百二十三 的字符串
 */
function numberToChinese(number) {
    var units = '个十百千万@#%亿^&~';
    var chars = '零一二三四五六七八九';
    var a = (number + '').split(''),
        s = [];
    if (a.length > 12) {
        throw new Error('too big');
    } else {
        for (var i = 0, j = a.length - 1; i <= j; i++) {
            if (j == 1 || j == 5 || j == 9) { //两位数 处理特殊的 1*
                if (i == 0) {
                    if (a[i] != '1') s.push(chars.charAt(a[i]));
                } else {
                    s.push(chars.charAt(a[i]));
                }
            } else {
                s.push(chars.charAt(a[i]));
            }
            if (i != j) {
                s.push(units.charAt(j - i));
            }
        }
    }
    return s.join('').replace(/零([十百千万亿@#%^&~])/g, function (m, d, b) { //优先处理 零百 零千 等
        b = units.indexOf(d);
        if (b != -1) {
            if (d == '亿') return d;
            if (d == '万') return d;
            if (a[j - b] == '0') return '零'
        }
        return '';
    }).replace(/零+/g, '零').replace(/零([万亿])/g, function (m, b) { // 零百 零千处理后 可能出现 零零相连的 再处理结尾为零的
        return b;
    }).replace(/亿[万千百]/g, '亿').replace(/[零]$/, '').replace(/[@#%^&~]/g, function (m) {
        return {
            '@': '十',
            '#': '百',
            '%': '千',
            '^': '十',
            '&': '百',
            '~': '千'
        }[m];
    }).replace(/([亿万])([一-九])/g, function (m, d, b, c) {
        c = units.indexOf(d);
        if (c != -1) {
            if (a[j - c] == '0') return d + '零' + b
        }
        return m;
    });
}

/**
 * 日期转中文，1-7 周一 - 周日
 * @param weekType 类型 zhweek：星期，其它： 周
 * @param val 周数 1-7（周一 - 周日）
 * @returns {String} 星期一/周一
 */
function zhWeek(weekType, val) {
    if (weekType == 'zhweek') {
        weekType = '星期';
    } else {
        weekType = '周';
    }
    if (val == 7) {
        return weekType + "日";
    } else {
        return weekType + numberCharToChinese(val);
    }
}
```