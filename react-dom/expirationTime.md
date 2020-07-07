# expirationTime

这个概念在源码中会到处看到，所以这里对相关的一些内容单独拿出来说一下

## requestCurrentTime

这个会请求一个时间可以理解为*当前时间-第一次加载时记录的时间*也就是打开页面的时间，但在渲染中我们请求的时间是一样的，这个就保证了我们多次调用，在很短的时间内得到的优先级是一样的，保证了在同一个时间段内更新

```js
function requestCurrentTime() {
  if (isRendering) {// 已经在渲染，返回之前计算的结果
    // We're already rendering. Return the most recently read time.
    return currentSchedulerTime;
  }
  // Check if there's pending work.
  findHighestPriorityRoot();// 找到优先级任务最高的root一般场景下只有一个
  if (
    nextFlushedExpirationTime === NoWork ||
    nextFlushedExpirationTime === Never
  ) {// 如果当前没有任务要做
    // If there's no pending work, or if the pending work is offscreen, we can
    // read the current time without risk of tearing.
    recomputeCurrentRendererTime(); // 从新计算时间
    currentSchedulerTime = currentRendererTime;
    return currentSchedulerTime;
  }
  // There's already pending work. We might be in the middle of a browser
  // event. If we were to read the current time, it could cause multiple updates
  // within the same event to receive different expiration times, leading to
  // tearing. Return the last read time. During the next idle callback, the
  // time will be updated.
  return currentSchedulerTime;
}

function recomputeCurrentRendererTime() {
  const currentTimeMs = now() - originalStartTimeMs;
  currentRendererTime = msToExpirationTime(currentTimeMs);
}
```

针对ExpirationTime涉及到很多计算不方便理解直接将源码拷贝出来，除去了不支持的语法，可以直接拷贝到控制台运行

```js
/**
 * Copyright (c) Facebook, Inc. and its affiliates.
 *
 * This source code is licensed under the MIT license found in the
 * LICENSE file in the root directory of this source tree.
 *
 * @flow
 */
const __DEV__=  false
const MAX_SIGNED_31_BIT_INT = 1073741823
const NoWork = 0;
const Never = 1;
const Sync = MAX_SIGNED_31_BIT_INT;

const UNIT_SIZE = 10;
const MAGIC_NUMBER_OFFSET = MAX_SIGNED_31_BIT_INT - 1;

// 1 unit of expiration time represents 10ms.
function msToExpirationTime(ms) {
  // Always add an offset so that we don't clash with the magic number for NoWork.
  return MAGIC_NUMBER_OFFSET - ((ms / UNIT_SIZE) | 0);
}

function expirationTimeToMs(expirationTime) {
  return (MAGIC_NUMBER_OFFSET - expirationTime) * UNIT_SIZE;
}

function ceiling(num, precision) {
  return (((num / precision) | 0) + 1) * precision;
}

function computeExpirationBucket(
  currentTime,
  expirationInMs,
  bucketSizeMs,
) {
  return (
    MAGIC_NUMBER_OFFSET -
    ceiling(
      MAGIC_NUMBER_OFFSET - currentTime + expirationInMs / UNIT_SIZE,
      bucketSizeMs / UNIT_SIZE,
    )
  );
}

const LOW_PRIORITY_EXPIRATION = 5000;
const LOW_PRIORITY_BATCH_SIZE = 250;

function computeAsyncExpiration(
  currentTime,
) {
  return computeExpirationBucket(
    currentTime,
    LOW_PRIORITY_EXPIRATION,
    LOW_PRIORITY_BATCH_SIZE,
  );
}

// We intentionally set a higher expiration time for interactive updates in
// dev than in production.
//
// If the main thread is being blocked so long that you hit the expiration,
// it's a problem that could be solved with better scheduling.
//
// People will be more likely to notice this and fix it with the long
// expiration time in development.
//
// In production we opt for better UX at the risk of masking scheduling
// problems, by expiring fast.
const HIGH_PRIORITY_EXPIRATION = __DEV__ ? 500 : 150;
const HIGH_PRIORITY_BATCH_SIZE = 100;

function computeInteractiveExpiration(currentTime) {
  return computeExpirationBucket(
    currentTime,
    HIGH_PRIORITY_EXPIRATION,
    HIGH_PRIORITY_BATCH_SIZE,
  );
}

computeInteractiveExpiration(1000) // 982
computeInteractiveExpiration(1007) // 982
computeInteractiveExpiration(1010) // 992
computeAsyncExpiration(1000) // 497
computeAsyncExpiration(1020) // 497

```

看最后几个调用方法`computeInteractiveExpiration`属于高优先级任务，`computeAsyncExpiration`是普通异步任务，计算出来的值是越大优先级越高，还有一点就是当传入的参数差距比较小时，得出来的结果是一样的，这也是保证前后2次调用更新但时间相差很短得到的优先级是一样的。主要是下边这个方法实现的,有一个取整的过程。我认为理解到这里就可以了。

```JS
function ceiling(num, precision) {
  return (((num / precision) | 0) + 1) * precision;
}
```