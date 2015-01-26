---
layout: default
---


之前在看RAC的时候，一直不太明白`doNext`方法，看官方文档上写的是：

	/// Do the given block on `next`. This should be used to inject side effects into
	/// the signal.
	- (RACSignal *)doNext:(void (^)(id x))block;
	
翻译成汉语就是“在`next`上执行`block`。会对`signal`产生副作用”。感觉还是不懂。直到今天，无意当中搜到limboy的博客上面对一段代码的注释：

	doNext:^(HBUser *user) {
	// doNext相当于一个钩子，是在sendNext时会被执行的一段代码
	                  @strongify(self);
	                  self.user = user;
	              }]

看到这个注释突然好像明白了什么，最近看书也渐渐有看源码的意识了。立马到源码里看了下。

	- (RACSignal *)doNext:(void (^)(id x))block {
		NSCParameterAssert(block != NULL);
	
		return [[RACSignal createSignal:^(id<RACSubscriber> subscriber) {
			return [self subscribeNext:^(id x) {
				block(x);
				[subscriber sendNext:x];
			} error:^(NSError *error) {
				[subscriber sendError:error];
			} completed:^{
				[subscriber sendCompleted];
			}];
		}] setNameWithFormat:@"[%@] -doNext:", self.name];
	}
可以只看这一小段代码：

	return [self subscribeNext:^(id x) {
					block(x);
					[subscriber sendNext:x];
				}
				
看这里已经很清楚了，`doNext`的`block`是在`sendNext`之前执行的。可以把limboy那段注释改为“**doNext相当于一个钩子，是在sendNext之前会被执行的一段代码**”