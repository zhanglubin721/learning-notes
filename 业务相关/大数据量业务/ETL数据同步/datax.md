在使用DataX进行数据同步时，如果你需要自定义一个Writer插件，你需要遵循DataX的Writer插件开发规范。以下是一个简单的自定义Writer示例：

1. 创建一个Maven项目，并添加DataX的依赖。
2. 实现`com.alibaba.datax.plugin.writer.common.Writer`接口。

以下是一个简单的自定义Writer的示例代码：

```java
package com.custom.datax.plugin.writer.customwriter;
 
import com.alibaba.datax.common.element.Record;
import com.alibaba.datax.common.exception.DataXException;
import com.alibaba.datax.common.plugin.RecordReceiver;
import com.alibaba.datax.common.plugin.TaskPluginCollector;
import com.alibaba.datax.plugin.writer.common.Writer;
import java.util.List;
 
public class CustomWriter extends Writer {
 
    // 省略其他必要的配置项和方法
 
    @Override
    public void write(RecordReceiver lineReceiver) throws Exception {
        // 实现数据写入逻辑
        Record record = null;
        while ((record = lineReceiver.getFromReader()) != null) {
            // 获取每一行的数据
            List<Object> columnValues = record.getColumnValues();
            // 这里只是打印出来，实际应用中应写入到你的数据存储系统
            System.out.println(columnValues);
        }
    }
 
    @Override
    public void post() {
        // 写入后的一些清理工作
    }
 
    @Override
    public void prepare() {
        // 写入前的准备工作
    }
 
    // 省略其他可能需要的方法
}
```

```json
{
    "job": {
        "setting": {
            "speed": {
                // ...
            },
            "errorLimit": {
                // ...
            }
        },
        "content": [
            {
                "reader": {
                    // ...
                },
                "writer": {
                    "name": "com.custom.datax.plugin.writer.customwriter.CustomWriter",
                    // 其他writer相关配置项
                    "parameter": {
                        // ...
                    }
                }
            }
        ]
    }
}
```

在这个例子中，你需要将`com.custom.datax.plugin.writer.customwriter.CustomWriter`替换为你自己的Writer类的全限定名。

请注意，这只是一个简单的示例，实际的Writer实现可能会涉及到更复杂的逻辑，例如错误处理、性能优化、分区写入、事务支持等。





## 前言

在 DataX 中，writer 是数据同步过程中的一个核心组件，负责将数据写入到目标数据源。下面是对 DataX 中 writer 组件的源码分析：<br>

 Writer 接口定义：<br>

 DataX 的 writer 组件首先定义了一个 Writer 接口，该接口定义了 writer 需要实现的基本方法，如 init(), write(), post() 等。<br>

 不同的数据源插件需要实现这个接口，提供对应的数据写入逻辑。<br>

 Writer 插件实现：<br>

 对于每种目标数据源，DataX 都会有一个对应的 writer 插件实现。例如，对于 MySQL 数据源，会有一个 MysqlWriter 类实现 Writer 接口。<br>

 每个 writer 插件的实现中，会包含与目标数据源交互的逻辑，如建立连接、执行 SQL 语句、批量插入数据等。<br>

 Writer 配置：<br>

 在 DataX 的 JSON 配置文件中，会指定 writer 的类型和相应的配置参数。<br>

 这些配置参数会被传递给 writer 插件的 init() 方法，用于初始化 writer 实例。<br>

 数据写入逻辑：<br>

 在 write() 方法中，writer 会从上游的 reader 中获取数据，并将其写入到目标数据源。<br>

 根据不同的数据源和写入策略，writer 可能会采用批量插入、逐条插入等方式进行数据写入。<br>

 writer 还会处理写入过程中的异常和错误，确保数据的完整性和一致性。<br>

 Writer 清理和关闭：<br>

 在数据写入完成后，writer 会执行 post() 方法，进行一些清理和关闭操作。<br>

 这可能包括关闭数据库连接、释放资源等。<br>

 通过对 DataX 中 writer 组件的源码分析，我们可以了解到 writer 是如何与目标数据源进行交互的，以及它是如何处理和写入数据的。

---

## DataX的Writer写入流程

- 初始化和准备：<br>

 根据配置文件中指定的目标数据源类型和参数，初始化Writer实例。<br>

 建立与目标数据源的连接，这通常涉及到网络连接、认证授权等步骤。<br>

 准备写入操作所需的各种资源，如缓冲区、事务等。
- 数据接收：<br>

 Writer从上游的Reader组件接收数据。这些数据可能是经过转换和处理的，已经符合目标数据源的要求。Writer将数据暂存到本地缓冲区或内存中，等待批量写入或逐条写入。
- 数据格式化和处理：<br>

 根据目标数据源的要求，Writer可能需要对接收到的数据进行格式化处理，如将数据转换为特定的文本格式、二进制格式或JSON格式等。
- 数据写入：<br>

 Writer将格式化处理后的数据写入目标数据源。写入操作可能涉及到网络通信、数据库操作等。根据目标数据源的特性，Writer会采用批量写入、流式写入等不同的写入方式以提高性能。对于支持事务的数据源，Writer会在每个写入操作前开启一个事务，并在写入完成后提交事务以确保数据的一致性。
- 错误处理和重试：<br>

 在写入过程中，Writer需要处理可能出现的各种错误和异常，如网络中断、数据格式错误等。根据配置文件中指定的错误处理策略，Writer可能会进行重试、跳过错误数据、记录错误日志等操作。
- 写入完成和清理：<br>

 当所有数据都成功写入目标数据源后，Writer会执行一些清理操作，如关闭数据库连接、释放资源等。Writer还会向上游的Reader或整个DataX任务发送完成信号，以通知整个任务流程已经完成。

<!-- -->

## Writer组件如何处理各类数据源

不同的数据源具有不同的写入特性和要求，因此Writer组件需要针对不同的数据源实现相应的写入逻辑。以下是一般情况下，DataX Writer组件如何处理各类数据源的大致步骤和考虑因素：

- 数据源连接：<br>

 Writer组件首先需要与目标数据源建立连接。这可能涉及到网络通信、认证授权、连接池管理等操作。<br>

 根据数据源类型的不同，Writer可能会使用不同的连接协议和库，如JDBC、ODBC、API等。
- 写入前准备：<br>

 根据目标数据源的表结构，Writer可能需要创建表、索引或分区。<br>

 Writer可能还需要准备写入数据的格式，如文本、二进制、JSON等。<br>

 对于支持事务的数据源，Writer可能会开启一个事务来确保数据的一致性。
- 数据写入：<br>

 Writer从Reader组件接收数据，并将其写入目标数据源。<br>

 根据数据源的特点，Writer可能会采用批量写入、逐条写入、流式写入等不同的写入方式。对于一些支持并行写入的数据源，Writer可能需要将数据分片并分配给多个线程或进程进行并发写入。
- 错误处理：<br>

 Writer需要处理写入过程中可能出现的异常和错误，如网络中断、数据格式错误、数据冲突等。<br>

 根据不同的错误类型，Writer可能会采取重试、跳过、记录错误日志等不同的处理策略。
- 写入优化：<br>

 对于不同的数据源，Writer可能会采用不同的优化策略来提高写入性能，如使用批量插入、调整事务大小、优化网络传输等。<br>

 Writer还可能利用目标数据源的特定功能，如批量提交、索引优化等，来进一步提高写入效率。
- 写入后处理：<br>

 在数据写入完成后，Writer可能会执行一些后处理操作，如提交事务、关闭连接、清理临时文件等。<br>

 对于一些需要额外处理的数据源，Writer可能还会执行数据校验、更新统计信息等操作。
- 扩展性和灵活性：<br>

 DataX的Writer组件设计通常具有高度的扩展性和灵活性，以便支持新的数据源类型。通过实现统一的接口和抽象类，可以方便地添加新的Writer插件来支持新的数据源。

<!-- -->

总之，DataX的Writer组件通过针对不同数据源实现特定的写入逻辑和优化策略，能够高效地处理各类数据源，并确保数据的正确性和一致性。同时，其扩展性和灵活性的设计也使得DataX能够轻松应对不断变化的数据处理需求。

## writer相关源码

```java
/**
 * 每个Writer插件需要实现Writer类，并在其内部实现Job、Task两个内部类。
 * 
 * 
 * */
public abstract class Writer extends BaseObject {
	/**
	 * 每个Writer插件必须实现Job内部类
	 */
	public abstract static class Job extends AbstractJobPlugin {
		/**
		 * 切分任务。<br>
		 * 
		 * @param mandatoryNumber
		 *            为了做到Reader、Writer任务数对等，这里要求Writer插件必须按照源端的切分数进行切分。否则框架报错！
		 * 
		 * */
		public abstract List<Configuration> split(int mandatoryNumber);
	}

	/**
	 * 每个Writer插件必须实现Task内部类
	 */
	public abstract static class Task extends AbstractTaskPlugin {

		public abstract void startWrite(RecordReceiver lineReceiver);

		public boolean supportFailOver(){return false;}
	}
}
```




```java
public class MysqlWriter extends Writer {
    private static final DataBaseType DATABASE_TYPE = DataBaseType.MySql;

    public static class Job extends Writer.Job {
        private Configuration originalConfig = null;
        private CommonRdbmsWriter.Job commonRdbmsWriterJob;

        @Override
        public void preCheck(){
            this.init();
            this.commonRdbmsWriterJob.writerPreCheck(this.originalConfig, DATABASE_TYPE);
        }

        @Override
        public void init() {
            this.originalConfig = super.getPluginJobConf();
            this.commonRdbmsWriterJob = new CommonRdbmsWriter.Job(DATABASE_TYPE);
            this.commonRdbmsWriterJob.init(this.originalConfig);
        }

        // 一般来说，是需要推迟到 task 中进行pre 的执行（单表情况例外）
        @Override
        public void prepare() {
            //实跑先不支持 权限 检验
            //this.commonRdbmsWriterJob.privilegeValid(this.originalConfig, DATABASE_TYPE);
            this.commonRdbmsWriterJob.prepare(this.originalConfig);
        }

        @Override
        public List<Configuration> split(int mandatoryNumber) {
            return this.commonRdbmsWriterJob.split(this.originalConfig, mandatoryNumber);
        }

        // 一般来说，是需要推迟到 task 中进行post 的执行（单表情况例外）
        @Override
        public void post() {
            this.commonRdbmsWriterJob.post(this.originalConfig);
        }

        @Override
        public void destroy() {
            this.commonRdbmsWriterJob.destroy(this.originalConfig);
        }

    }

    public static class Task extends Writer.Task {
        private Configuration writerSliceConfig;
        private CommonRdbmsWriter.Task commonRdbmsWriterTask;

        @Override
        public void init() {
            this.writerSliceConfig = super.getPluginJobConf();
            this.commonRdbmsWriterTask = new CommonRdbmsWriter.Task(DATABASE_TYPE);
            this.commonRdbmsWriterTask.init(this.writerSliceConfig);
        }

        @Override
        public void prepare() {
            this.commonRdbmsWriterTask.prepare(this.writerSliceConfig);
        }

        //TODO 改用连接池，确保每次获取的连接都是可用的（注意：连接可能需要每次都初始化其 session）
        public void startWrite(RecordReceiver recordReceiver) {
            this.commonRdbmsWriterTask.startWrite(recordReceiver, this.writerSliceConfig,
                    super.getTaskPluginCollector());
        }

        @Override
        public void post() {
            this.commonRdbmsWriterTask.post(this.writerSliceConfig);
        }

        @Override
        public void destroy() {
            this.commonRdbmsWriterTask.destroy(this.writerSliceConfig);
        }

        @Override
        public boolean supportFailOver(){
            String writeMode = writerSliceConfig.getString(Key.WRITE_MODE);
            return "replace".equalsIgnoreCase(writeMode);
        }

    }
}
```

```java
public class RdbmsWriter extends Writer {
    private static final DataBaseType DATABASE_TYPE = DataBaseType.RDBMS;
    static {
    	//加载插件下面配置的驱动类
        DBUtil.loadDriverClass("writer", "rdbms");
    }
    public static class Job extends Writer.Job {
        private Configuration originalConfig = null;
        private CommonRdbmsWriter.Job commonRdbmsWriterMaster;

        @Override
        public void init() {
            this.originalConfig = super.getPluginJobConf();

            // warn：not like mysql, only support insert mode, don't use
            String writeMode = this.originalConfig.getString(Key.WRITE_MODE);
            if (null != writeMode) {
                throw DataXException
                        .asDataXException(
                                DBUtilErrorCode.CONF_ERROR,
                                String.format(
                                        "写入模式(writeMode)配置有误. 因为不支持配置参数项 writeMode: %s, 仅使用insert sql 插入数据. 请检查您的配置并作出修改.",
                                        writeMode));
            }

            this.commonRdbmsWriterMaster = new SubCommonRdbmsWriter.Job(
                    DATABASE_TYPE);
            this.commonRdbmsWriterMaster.init(this.originalConfig);
        }

        @Override
        public void prepare() {
            this.commonRdbmsWriterMaster.prepare(this.originalConfig);
        }

        @Override
        public List<Configuration> split(int mandatoryNumber) {
            return this.commonRdbmsWriterMaster.split(this.originalConfig,
                    mandatoryNumber);
        }

        @Override
        public void post() {
            this.commonRdbmsWriterMaster.post(this.originalConfig);
        }

        @Override
        public void destroy() {
            this.commonRdbmsWriterMaster.destroy(this.originalConfig);
        }

    }

    public static class Task extends Writer.Task {
        private Configuration writerSliceConfig;
        private CommonRdbmsWriter.Task commonRdbmsWriterSlave;

        @Override
        public void init() {
            this.writerSliceConfig = super.getPluginJobConf();
            this.commonRdbmsWriterSlave = new SubCommonRdbmsWriter.Task(
                    DATABASE_TYPE);
            this.commonRdbmsWriterSlave.init(this.writerSliceConfig);
        }

        @Override
        public void prepare() {
            this.commonRdbmsWriterSlave.prepare(this.writerSliceConfig);
        }

        public void startWrite(RecordReceiver recordReceiver) {
            this.commonRdbmsWriterSlave.startWrite(recordReceiver,
                    this.writerSliceConfig, super.getTaskPluginCollector());
        }

        @Override
        public void post() {
            this.commonRdbmsWriterSlave.post(this.writerSliceConfig);
        }

        @Override
        public void destroy() {
            this.commonRdbmsWriterSlave.destroy(this.writerSliceConfig);
        }
    }
}
```







```json
{
  "name": "printwriter",
  "developer": "",
  "description": "print writer plugin",
  "version": "1.0",
  "encoding": "utf-8"
}
```

PrintWriterJob.java 是提交作业时的主类，主要用来解析用户配置和校验用户输入：

```java
package com.alibaba.datax.plugin.writer.printwriter;

import com.alibaba.datax.common.element.Record;
import com.alibaba.datax.common.plugin.JobPluginCollector;
import com.alibaba.datax.common.plugin.RecordReceiver;
import com.alibaba.datax.common.spi.Writer;

import java.io.IOException;

public class PrintWriterJob extends Writer {
    // 校验配置文件等其他入口性检查
    @Override
    public void prepare(JobPluginCollector jobPluginCollector)  {
        // TODO 
    }

    // 将用户配置切分成多块
    @Override
    public void split(int mandatoryNumber) {
        // TODO
    }
}
```

PrintWriterTask.java 是最终写出数据的主类:

```java
package com.alibaba.datax.plugin.writer.printwriter;

import com.alibaba.datax.common.element.Record;
import com.alibaba.datax.common.plugin.TaskPluginCollector;
import com.alibaba.datax.common.spi.Writer;

import java.io.IOException;

public class PrintWriterTask extends Writer {
    // 开始写入前准备资源
    public void startWrite(RecordReceiver lineReceiver, TaskPluginCollector collector) {
        try {
            Record record;
            while ((record = lineReceiver.getFromReader()) != null) {
                //打印数据
                System.out.println(record.toString());
                 
                //错误处理
                try {
                    //写入逻辑
                    //TODO
                } catch (Exception e) {
                    collector.collectDirtyRecord(record,e);
                }
            }
            //优化代码以保持良好的性能
            //TODO
        } finally {
            //释放资源
            //TODO
        }
    }

    //异常处理
    public void post() {
    }

    //撤销操作
    public void destroy() {
    }
}
```

以上是一个基础版本的实现，为了分区写入和事务支持，我们可能需要更复杂的代码逻辑。例如，我们可以考虑使用数据库的分区功能，并在post方法实现COMMIT操作，在destroy方法实现ROLLBACK操作，这便视具体的数据源或目标系统而定。