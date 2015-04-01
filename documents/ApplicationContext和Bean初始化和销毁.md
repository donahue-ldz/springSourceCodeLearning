* 对于BeanFactory,特别是ApplicationContext 容器自身也是有一个初始化和销毁的过程
* ApplicationContext启动的过程是在abstractApplicationContext中实现的，使用上下文的时候需要做一些准备准备工作是在prepareBeanFactory()中实现，在这个方法中为容器配置ClassLoader，PropertyEditor和BeanPostProcessor等为容器的启动做好充分的准备
* 当容器需要关闭的时候，也需要完成一系列的工作，doClose() 首先发出关闭的事件信号，然后在将bean逐个关闭。最后关闭容器本身
具体代码参考

```
protected void doClose() {
		boolean actuallyClose;
		synchronized (this.activeMonitor) {
			actuallyClose = this.active && !this.closed;
			this.closed = true;
		}

		if (actuallyClose) {
			if (logger.isInfoEnabled()) {
				logger.info("Closing " + this);
			}

			LiveBeansView.unregisterApplicationContext(this);

			try {
				// Publish shutdown event.
				publishEvent(new ContextClosedEvent(this));
			}
			catch (Throwable ex) {
				logger.warn("Exception thrown from ApplicationListener handling ContextClosedEvent", ex);
			}

			// Stop all Lifecycle beans, to avoid delays during individual destruction.
			try {
				getLifecycleProcessor().onClose();
			}
			catch (Throwable ex) {
				logger.warn("Exception thrown from LifecycleProcessor on context close", ex);
			}

			// Destroy all cached singletons in the context's BeanFactory.
			destroyBeans();

			// Close the state of this context itself.
			closeBeanFactory();

			// Let subclasses do some final clean-up if they wish...
			onClose();

			synchronized (this.activeMonitor) {
				this.active = false;
			}
		}
	}
	
	
	```
