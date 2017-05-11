## Java Security Code Snippet<br>
java安全相关的内容学习和代码块记录。虽然在开发中直接遇到的java应用程序安全方面的内容不多，整体来说，java应用还是非常安全的。不过也总会有些安全问题出现，我就当做学习了解记录下来<br>

* 简单的防止Java程序调用外部系统命令<br>
  在微信公众号推荐中看到的一篇，关于JVM调用外部命令造成的漏洞安全问题。详情参考:[禁止JVM执行外部命令Runtime.exec -- 由Apache Commons Collections漏洞引发的思考](http://blog.csdn.net/hengyunabc/article/details/49804577)<br>
  那么就直接将简单防止JVM调用外部命令的代码块列出，可以在日后若是有相关问题，作为借鉴与思考:<br>
```java
  SecurityManager originalSecurityManager = System.getSecurityManager();
        if (originalSecurityManager == null) {
            // 创建自己的SecurityManager
            SecurityManager sm = new SecurityManager() {
                private void check(Permission perm) {
                    // 禁止exec
                    if (perm instanceof java.io.FilePermission) {
                        String actions = perm.getActions();
                        if (actions != null && actions.contains("execute")) {
                            throw new SecurityException("execute denied!");
                        }
                    }
                    // 禁止设置新的SecurityManager，保护自己
                    if (perm instanceof java.lang.RuntimePermission) {
                        String name = perm.getName();
                        if (name != null && name.contains("setSecurityManager")) {
                            throw new SecurityException("System.setSecurityManager denied!");
                        }
                    }
                }

                @Override
                public void checkPermission(Permission perm) {
                    check(perm);
                }

                @Override
                public void checkPermission(Permission perm, Object context) {
                    check(perm);
                }
            };

            System.setSecurityManager(sm);
        }
```
