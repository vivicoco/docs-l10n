# 백엔드

[TOC]

A backend is the composition of a [compiler](compilation.md#compiler) and a [runtime](execution.md#runtime) in a [Context](context.md#context) used to [construct](tracing.md), [compile](compilation.md), and [execute](execution.md) an [AST](compilation.md#ast), meaning a backend constructs environments that evaluate an AST.

The [backends](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends) package contains backends which may extend the TFF compiler and/or the TFF runtime; these extensions can be found in the corresponding backend.

If the [runtime](execution.md#runtime) of a backend is implemented as an [execution stack](execution.md#execution-stack), then the backend can construct an [ExecutionContext](context.md#executioncontext) to provide TFF with an environemnt in which to evaluate an AST. In this case, the backend is integrating with TFF using the high-level abstraction. However, if the runtime is *not* implemented as an execution stack, then the backend will need to construct a [Context](context.md#context) and is integrating with TFF using the low-level abstraction.

```dot
<!--#include file="backend.dot"-->
```

The **blue** nodes are provided by TFF [core](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core).

The **green**, **red**, **yellow**, and **purple** nodes are provided by the [native](#native), [mapreduce](#mapreduce), [iree](#iree), and [reference](#reference) backends respectively.

The **dashed** nodes are provided by an external system.

**실선** 화살표는 관계를 나타내고 **점선** 화살표는 상속을 나타냅니다.

## Native

The [native](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/native) backend composes of the TFF compiler and TFF runtime in order to compile and execute an AST in a way that is reasonably efficiant and debuggable.

### 네이티브 형식

A native form is an AST that is topologically sorted into a directed acyclic graph (DAG) of TFF intrinsics with some optimizations to the dependency of those intrinsics.

### 컴파일러

The [compiler.transform_to_native_form](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/native/compiler.py) function compiles an AST into a [native form](#native-form).

### Runtime

The native backend does not contain backend specific extentions to the TFF runtime, instead an [execution stack](execution.md#execution-stack) can be used directly.

### Context

A native context is an [ExecutionContext](context.md#executioncontext) constructed with a native compiler (or no compiler) and a TFF runtime, for example:

```python
executor = eager_tf_executor.EagerTFExecutor()
factory = executor_factory.create_executor_factory(lambda _: executor)
context = execution_context.ExecutionContext(
    executor_fn=factory,
    compiler_fn=None)
set_default_context.set_default_context(context)
```

그러나 몇 가지 일반적인 구성이 있습니다.

The [execution_context.set_local_execution_context](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/native/execution_context.py) function constructs an `ExecutionContext` with a native compiler and a [local execution stack](execution.md#local-execution-stack).

## MapReduce

The [mapreduce](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/mapreduce) backend contains the data structures and compiler required to construct a form that can be executed on MapReduce-like runtimes.

### `CanonicalForm`

A [forms.CanonicalForm](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/mapreduce/forms.py) is a data structure defining the representation of logic that can be executed on MapReduce-like runtimes. This logic is organized as a collection of TensorFlow functions, see the [forms](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/mapreduce/forms.py) module for more information about the nature of these functions.

### 컴파일러

The [transformations](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/mapreduce/transformations.py) module contains [Building Block](compilation.md#building-block) and [TensorFlow Computation](compilation.md#tensorflow-computation) transformations required to compile an AST to a [CanonicalForm](#canonicalform).

The [form_utils](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/mapreduce/form_utils.py) module contains the compiler for the MapReduce backend and constructs a [CanonicalForm](#canonicalform).

### Runtime

A MapReduce runtime is not provided by TFF, instead this should be provided by an external MapReduce-like system.

### Context

MapReduce 컨텍스트는 TFF에서 제공하지 않습니다.

## IREE

[IREE](https://github.com/google/iree) is an experimental compiler backend for [MLIR](https://mlir.llvm.org/).

[iree](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/iree) 백엔드에는 AST를 실행하는 데 필요한 데이터 구조, 컴파일러 및 런타임이 포함됩니다.

### 컴파일러

The [compiler](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/iree/compiler.py) module contains transformations required to comiple an AST to a form that can be exected using an [executor.IreeExecutor](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/iree/executor.py).

### Runtime

The [executor.IreeExecutor](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/iree/executor.py) is an [Executor](execution.md#executor) that executes computations by delegating to an IREE runtime. This executor can be composed with other [Executors](execution.md#executor) from the TFF runtime in order to construct an [execution stack](execution.md#execution-stack) representing an IREE runtime.

### Context

An iree context is [ExecutionContext](context.md#executioncontext) constructed with an iree compiler and an [execution stack](execution.md#execution-stack) with an [executor.IreeExecutor](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/iree/executor.py) delegating to an external IREE runtime.

## 참고

A [reference_context.ReferenceContext](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/backends/reference/reference_context.py) is a [context_base.Context](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/impl/context_stack/context_base.py) that compiles and executes ASTs. Note that the `ReferenceContext` does not inherit from [execution_context.ExecutionContext](https://github.com/tensorflow/federated/blob/master/tensorflow_federated/python/core/impl/executors/execution_context.py) and the runtime is not implemented as an [execution stack](execution.md#execution-stack); instead the compiler and runtime are trivially implemented inline in the `ReferenceContext`.