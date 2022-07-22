面对这样的`if`语句，你是不是很难受呢？

```java
if (flag == 1) {
  log.info("didispace.com: 1");
} else if (flag == 2) {
  log.info("didispace.com: 2");
} else if (flag == 3) {
  log.info("didispace.com: 3");
} else if (flag == 4) {
  log.info("didispace.com: 4");
} else {
  log.info("didispace.com: x");
}
```

是不是想到用`switch`来改进一下？

```java
switch(flag) {
  case 1: 
    log.info("didispace.com: 1"); 
    break;
  case 2:
    log.info("didispace.com: 2");
    break;
  case 3:
    log.info("didispace.com: 3");
    break;
  case 4:
    log.info("didispace.com: 4");
    break;
  default:
    log.info("didispace.com: x");
}
```

舒服了吗？是不是感觉还是不那么舒服呢？

试试Java 14中对Switch表达式的增强功能，继续改造：

```java
switch(flag) {
  case 1  -> log.info("didispace.com: 1");
  case 2  -> log.info("didispace.com: 2");
  case 3  -> log.info("didispace.com: 3");
  case 4  -> log.info("didispace.com: 4");
  default -> log.info("didispace.com: x");
}
```

这下是不是舒服了？在Java 14的switch表达式增强中，引入了对Lambda语法的支持，让每个case分支变得更为简洁。同时，容易遗忘的`break`也可以省略了。