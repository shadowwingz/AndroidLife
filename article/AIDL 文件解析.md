```java
public interface IBookManager extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    // 声明了一个内部类 Stub，这个 Stub 就是一个 Binder 类
    // 这个类，继承了 Binder，实现了 IBookManager
    // 调用 asInterface 时，它可以调用 IBookManager 的跨进程方法
    // 调用 asBinder 时，它可以关联 Binder 的死亡代理
    public static abstract class Stub extends android.os.Binder implements com.example
            .chapter2first.aidl.IBookManager {
        // Binder 的唯一标识，一般用当前 Binder 的类名表示
        private static final java.lang.String DESCRIPTOR = "com.example.chapter2first.aidl" +
                ".IBookManager";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.example.chapter2first.aidl.IBookManager interface,
         * generating a proxy if needed.
         */
        // 用于将服务端的 Binder 对象转换成客户端所需的 AIDL 接口类型的对象，这种
        // 转换过程是区分进程的，如果客户端和服务端位于同一进程，那么此方法返回的就是
        // 服务端的 Stub 对象本身，否则返回的是系统封装后的 Stub.proxy 对象
        public static com.example.chapter2first.aidl.IBookManager asInterface(android.os.IBinder
                                                                                      obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.example.chapter2first.aidl.IBookManager))) {
                // 不跨进程
                return ((com.example.chapter2first.aidl.IBookManager) iin);
            }
            // 跨进程
            return new com.example.chapter2first.aidl.IBookManager.Stub.Proxy(obj);
        }

		// 此方法用于返回当前 Binder 对象
        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

		// 当跨进程时，就会走这个方法，然后根据 id 调用相应的方法
		// 这个方法运行在服务端的 Binder 线程池中，当客户端发起跨进程请求时，
		// 远程请求会通过系统底层封装后交由此方法来处理。服务端通过 code 可以
		// 确定客户端所请求的目标方法是什么，接着从 data 中取出目标方法所需的参数
		// （如果目标方法有参数的话），然后执行目标方法。当目标方法执行完毕后，就向 reply
		// 中写入返回值（如果目标方法有返回值的话），onTransact 方法的执行过程就是这样的
		// 需要注意的是，如果此方法返回 false，那么客户端的请求会失败，因此我们
		// 可以利用这个特性来做权限验证
        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int
                flags) throws android.os.RemoteException {
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(DESCRIPTOR);
                    return true;
                }
                case TRANSACTION_getBookList: {
                    data.enforceInterface(DESCRIPTOR);
                    java.util.List<com.example.chapter2first.aidl.Book> _result = this
                            .getBookList();
                    reply.writeNoException();
                    reply.writeTypedList(_result);
                    return true;
                }
                case TRANSACTION_addBook: {
                    data.enforceInterface(DESCRIPTOR);
                    com.example.chapter2first.aidl.Book _arg0;
                    if ((0 != data.readInt())) {
                        _arg0 = com.example.chapter2first.aidl.Book.CREATOR.createFromParcel(data);
                    } else {
                        _arg0 = null;
                    }
                    this.addBook(_arg0);
                    reply.writeNoException();
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }

		// Proxy 里的 getBookList 和 addBook 方法并不是具体的实现，而是
		// 跨进程调用相应的方法，具体的实现是由开发者完成。
        private static class Proxy implements com.example.chapter2first.aidl.IBookManager {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

			// 这个方法运行在客户端，当客户端远程调用此方法时，它的内部实现是这样的：
			// 首先创建该方法所需要的输入型 Parcel 对象 _data、输出型 Parcel 对象 _reply
			// 和返回值对象 List，然后把该方法的参数信息写入 _data 中（如果有参数的话），
			// 接着调用 transact 方法来发起 RPC（远程过程调用），同时当前线程挂起，然后
			// 服务端的 onTransact 方法会被调用，直到 RPC 过程返回后，当前线程继续执行，
			// 并从 _reply 中取出 RPC 过程的返回结果，最后返回 _reply 中的数据
            @Override
            public java.util.List<com.example.chapter2first.aidl.Book> getBookList() throws
                    android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                java.util.List<com.example.chapter2first.aidl.Book> _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getBookList, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.createTypedArrayList(com.example.chapter2first.aidl.Book.CREATOR);
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }

			// 这个方法运行在客户端，它的执行过程和 getBookList 是一样的，addBook 没有返回值，
			// 所以它不需要从 _reply 中取出返回值
            @Override
            public void addBook(com.example.chapter2first.aidl.Book book) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    if ((book != null)) {
                        _data.writeInt(1);
                        book.writeToParcel(_data, 0);
                    } else {
                        _data.writeInt(0);
                    }
                    mRemote.transact(Stub.TRANSACTION_addBook, _data, _reply, 0);
                    _reply.readException();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }
        }

		// 声明了两个整型的 id 标识下面的两个方法
		// 这两个 id 用于标识在 transact 过程中客户端请求的
		// 到底是哪个方法
        static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }

	// 声明了两个方法 getBookList 和 addBook
	// 也就是我们在 IBookManager.aidl 中声明的方法
    public java.util.List<com.example.chapter2first.aidl.Book> getBookList() throws android.os.RemoteException;

    public void addBook(com.example.chapter2first.aidl.Book book) throws android.os.RemoteException;
}

```