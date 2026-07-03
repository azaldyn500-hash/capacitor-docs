<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>صيدليتك الإلكترونية - النظام المتكامل</title>
    
    <!-- React & ReactDOM -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js" crossorigin></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js" crossorigin></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.5/babel.min.js" crossorigin></script>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>

    <!-- Firebase SDKs -->
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-auth-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore-compat.js"></script>

    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700;900&display=swap" rel="stylesheet">

    <style>
        body { font-family: 'Cairo', sans-serif; background-color: #f8fafc; -webkit-tap-highlight-color: transparent; }
        .hide-scroll::-webkit-scrollbar { display: none; }
        .hide-scroll { -ms-overflow-style: none; scrollbar-width: none; }
        
        /* Loading Animation */
        .loader {
            border: 3px solid #f3f3f3; border-top: 3px solid #059669;
            border-radius: 50%; width: 24px; height: 24px; animation: spin 1s linear infinite;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useMemo, useRef } = React;

        // --- 1. Firebase Initialization (Canvas Compliant) ---
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {
            // Fallback for local testing if needed, but Canvas provides this
        };
        
        if (!firebase.apps.length) {
            firebase.initializeApp(firebaseConfig);
        }
        const auth = firebase.auth();
        const db = firebase.firestore();
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'pharmacy-app-dev';

        // Helper function for strict Canvas Firebase paths
        const getCollection = (collectionName) => {
            return db.collection('artifacts').doc(appId).collection('public').doc('data').collection(collectionName);
        };

        // --- 2. Main Application Component ---
        function App() {
            const [user, setUser] = useState(null);
            const [authLoading, setAuthLoading] = useState(true);
            const [role, setRole] = useState('user'); // user, admin, delivery
            
            // Global Data States
            const [medicines, setMedicines] = useState([]);
            const [orders, setOrders] = useState([]);
            const [cart, setCart] = useState([]);
            const [notifications, setNotifications] = useState([]);

            // --- Authentication Effect ---
            useEffect(() => {
                const initAuth = async () => {
                    try {
                        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                            await auth.signInWithCustomToken(__initial_auth_token);
                        } else {
                            await auth.signInAnonymously();
                        }
                    } catch (error) {
                        console.error("Auth Error:", error);
                    }
                };
                initAuth();

                const unsubscribe = auth.onAuthStateChanged((u) => {
                    setUser(u);
                    setAuthLoading(false);
                });
                return () => unsubscribe();
            }, []);

            // --- Data Fetching Effect (Real-time Firestore) ---
            useEffect(() => {
                if (!user) return;

                // 1. Fetch Medicines
                const unsubMeds = getCollection('medicines').onSnapshot(
                    (snapshot) => {
                        const meds = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                        setMedicines(meds.length > 0 ? meds : DEFAULT_MEDICINES); // Fallback if DB empty
                    },
                    (error) => console.error("Meds error:", error)
                );

                // 2. Fetch Orders
                const unsubOrders = getCollection('orders').onSnapshot(
                    (snapshot) => {
                        const ords = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                        // Client-side sorting (Canvas rule: no complex queries)
                        setOrders(ords.sort((a, b) => b.timestamp - a.timestamp)); 
                    },
                    (error) => console.error("Orders error:", error)
                );

                return () => { unsubMeds(); unsubOrders(); };
            }, [user]);

            const notify = (msg, type='success') => {
                const id = Date.now();
                setNotifications(p => [...p, {id, msg, type}]);
                setTimeout(() => setNotifications(p => p.filter(n => n.id !== id)), 3000);
            };

            if (authLoading) return <div className="min-h-screen flex items-center justify-center bg-slate-50"><div className="loader w-12 h-12"></div></div>;

            return (
                <div className="min-h-screen flex flex-col bg-slate-50 text-slate-800">
                    {/* Top Dev/Role Switcher (For Demo Purposes) */}
                    <div className="bg-slate-900 text-white text-[10px] p-2 flex justify-between items-center z-50 relative">
                        <span className="font-bold text-emerald-400">محاكي التطبيق - قيد التشغيل 🟢</span>
                        <div className="flex gap-2">
                            <button onClick={()=>setRole('user')} className={`px-2 py-1 rounded ${role==='user'?'bg-emerald-600':'bg-slate-700'}`}>واجهة العميل</button>
                            <button onClick={()=>setRole('admin')} className={`px-2 py-1 rounded ${role==='admin'?'bg-emerald-600':'bg-slate-700'}`}>الإدارة</button>
                            <button onClick={()=>setRole('delivery')} className={`px-2 py-1 rounded ${role==='delivery'?'bg-emerald-600':'bg-slate-700'}`}>المندوب</button>
                        </div>
                    </div>

                    {/* Main Content Area */}
                    <div className="flex-1 pb-20 overflow-y-auto">
                        {role === 'user' && <ClientApp user={user} medicines={medicines} cart={cart} setCart={setCart} orders={orders.filter(o => o.userId === user.uid)} notify={notify} db={getCollection} />}
                        {role === 'admin' && <AdminApp user={user} medicines={medicines} orders={orders} notify={notify} db={getCollection} />}
                        {role === 'delivery' && <DeliveryApp user={user} orders={orders.filter(o => ['ready', 'delivering'].includes(o.status))} notify={notify} db={getCollection} />}
                    </div>

                    {/* Notifications Toast */}
                    <div className="fixed bottom-24 left-4 right-4 z-50 flex flex-col gap-2 pointer-events-none">
                        {notifications.map(n => (
                            <div key={n.id} className={`p-3 rounded-lg shadow-lg text-xs font-bold text-white transition-all transform translate-y-0 opacity-100 ${n.type === 'error' ? 'bg-red-500' : 'bg-slate-800 border-r-4 border-emerald-500'}`}>
                                {n.msg}
                            </div>
                        ))}
                    </div>
                </div>
            );
        }

        // ==========================================
        // 1. CLIENT APP (Customer Interface)
        // ==========================================
        function ClientApp({ user, medicines, cart, setCart, orders, notify, db }) {
            const [activeTab, setActiveTab] = useState('home'); // home, cart, orders, profile
            const [search, setSearch] = useState('');
            const [category, setCategory] = useState('الكل');

            const categories = ['الكل', ...new Set(medicines.map(m => m.category))];
            const filteredMeds = medicines.filter(m => 
                (category === 'الكل' || m.category === category) &&
                (m.trade_name.toLowerCase().includes(search.toLowerCase()) || m.scientific_name.toLowerCase().includes(search.toLowerCase()))
            );

            const addToCart = (med) => {
                setCart(prev => {
                    const exists = prev.find(item => item.id === med.id);
                    if (exists) return prev.map(item => item.id === med.id ? {...item, qty: item.qty + 1} : item);
                    return [...prev, {...med, qty: 1}];
                });
                notify(`تم إضافة ${med.trade_name} للسلة`);
            };

            return (
                <div className="h-full flex flex-col relative">
                    {/* Header */}
                    <header className="bg-white p-4 shadow-sm sticky top-0 z-30">
                        <div className="flex justify-between items-center">
                            <div className="flex items-center gap-2">
                                <div className="w-8 h-8 bg-emerald-600 rounded-lg flex items-center justify-center text-white font-bold">+</div>
                                <div>
                                    <h1 className="font-black text-sm leading-none">صيدليتك</h1>
                                    <span className="text-[9px] text-emerald-600 font-bold">بضغطة زر دواؤك عندك</span>
                                </div>
                            </div>
                            {user && <span className="text-xs bg-slate-100 px-2 py-1 rounded text-slate-500">{user.uid.substring(0,5)}...</span>}
                        </div>
                    </header>

                    {/* Content */}
                    <div className="p-4">
                        {activeTab === 'home' && (
                            <div className="space-y-6">
                                {/* Search */}
                                <div className="relative">
                                    <input 
                                        type="text" placeholder="ابحث عن دواء، مسكن، فيتامين..." 
                                        value={search} onChange={e=>setSearch(e.target.value)}
                                        className="w-full bg-white border border-slate-200 p-3 rounded-2xl text-xs pr-10 outline-none focus:border-emerald-500 shadow-sm"
                                    />
                                    <svg className="w-5 h-5 absolute right-3 top-3 text-slate-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path></svg>
                                </div>

                                {/* Banner */}
                                <div className="bg-gradient-to-l from-emerald-600 to-teal-700 rounded-2xl p-5 text-white shadow-md relative overflow-hidden">
                                    <h2 className="font-black text-lg mb-1 relative z-10">ارفع روشتتك الان!</h2>
                                    <p className="text-[10px] text-emerald-100 relative z-10 max-w-[70%]">صور الوصفة الطبية وسنقوم بتجهيزها وإرسالها لك فوراً.</p>
                                    <button onClick={()=>setActiveTab('prescription')} className="mt-3 bg-white text-emerald-700 text-xs font-bold px-4 py-2 rounded-xl relative z-10 shadow-sm">ارفع الوصفة 📷</button>
                                    <div className="absolute left-[-20px] bottom-[-20px] text-8xl opacity-20">💊</div>
                                </div>

                                {/* Categories */}
                                <div className="flex gap-2 overflow-x-auto hide-scroll pb-2">
                                    {categories.map(cat => (
                                        <button key={cat} onClick={()=>setCategory(cat)} className={`whitespace-nowrap px-4 py-1.5 rounded-full text-xs font-bold transition-colors ${category === cat ? 'bg-emerald-600 text-white' : 'bg-white border border-slate-200 text-slate-600'}`}>
                                            {cat}
                                        </button>
                                    ))}
                                </div>

                                {/* Medicines Grid */}
                                <div className="grid grid-cols-2 gap-3">
                                    {filteredMeds.map(med => (
                                        <div key={med.id} className="bg-white rounded-2xl p-3 shadow-sm border border-slate-100 flex flex-col justify-between">
                                            <div>
                                                <div className="text-3xl mb-2 text-center bg-slate-50 rounded-xl py-4">{med.image_url}</div>
                                                <h3 className="font-black text-xs text-slate-800 line-clamp-1">{med.trade_name}</h3>
                                                <p className="text-[9px] text-slate-500 line-clamp-1">{med.scientific_name}</p>
                                            </div>
                                            <div className="mt-3 flex justify-between items-center">
                                                <span className="font-black text-emerald-600 text-sm">{med.price_yer} <span className="text-[9px]">ر.ي</span></span>
                                                <button onClick={()=>addToCart(med)} className="bg-slate-900 text-white w-7 h-7 rounded-lg flex items-center justify-center shadow-md hover:bg-emerald-600 transition-colors">+</button>
                                            </div>
                                        </div>
                                    ))}
                                </div>
                            </div>
                        )}

                        {activeTab === 'cart' && <CartView cart={cart} setCart={setCart} user={user} db={db} notify={notify} setActiveTab={setActiveTab} />}
                        {activeTab === 'orders' && <OrdersView orders={orders} />}
                        {activeTab === 'prescription' && <PrescriptionView user={user} db={db} notify={notify} setActiveTab={setActiveTab} />}
                    </div>

                    {/* Bottom Navigation */}
                    <nav className="bg-white border-t border-slate-200 fixed bottom-0 w-full flex justify-around p-3 pb-safe z-40 shadow-[0_-4px_6px_-1px_rgba(0,0,0,0.05)]">
                        <NavItem icon="🏠" label="الرئيسية" active={activeTab==='home'} onClick={()=>setActiveTab('home')} />
                        <NavItem icon="🛒" label="السلة" active={activeTab==='cart'} onClick={()=>setActiveTab('cart')} badge={cart.reduce((a,c)=>a+c.qty,0)} />
                        <NavItem icon="📋" label="طلباتي" active={activeTab==='orders'} onClick={()=>setActiveTab('orders')} badge={orders.filter(o=>o.status!=='completed').length} />
                        <NavItem icon="📷" label="روشتة" active={activeTab==='prescription'} onClick={()=>setActiveTab('prescription')} />
                    </nav>
                </div>
            );
        }

        function NavItem({ icon, label, active, onClick, badge }) {
            return (
                <button onClick={onClick} className={`flex flex-col items-center gap-1 relative ${active ? 'text-emerald-600' : 'text-slate-400'}`}>
                    <span className="text-xl relative">
                        {icon}
                        {badge > 0 && <span className="absolute -top-1 -right-2 bg-red-500 text-white text-[9px] font-bold w-4 h-4 flex items-center justify-center rounded-full border-2 border-white">{badge}</span>}
                    </span>
                    <span className={`text-[9px] font-bold ${active ? 'opacity-100' : 'opacity-70'}`}>{label}</span>
                </button>
            );
        }

        // --- Cart Component ---
        function CartView({ cart, setCart, user, db, notify, setActiveTab }) {
            const [address, setAddress] = useState('صنعاء - حدة');
            const [phone, setPhone] = useState('777xxxxxx');
            const total = cart.reduce((sum, item) => sum + (item.price_yer * item.qty), 0);
            const deliveryFee = 1000;

            const handleCheckout = async () => {
                if (cart.length === 0) return notify('السلة فارغة', 'error');
                
                const newOrder = {
                    userId: user.uid,
                    customerName: "عميل", // In real app, get from profile
                    phone,
                    address,
                    items: cart,
                    subtotal: total,
                    deliveryFee,
                    total: total + deliveryFee,
                    status: 'new',
                    timestamp: Date.now(),
                    type: 'standard'
                };

                try {
                    await db('orders').add(newOrder);
                    setCart([]);
                    notify('تم إرسال الطلب بنجاح! جاري التجهيز.');
                    setActiveTab('orders');
                } catch (e) {
                    notify('حدث خطأ أثناء الطلب', 'error');
                    console.error(e);
                }
            };

            return (
                <div className="space-y-4">
                    <h2 className="font-black text-lg border-b pb-2">سلة المشتريات</h2>
                    {cart.length === 0 ? (
                        <div className="text-center py-10 text-slate-400">السلة فارغة</div>
                    ) : (
                        <div className="space-y-4">
                            <div className="bg-white rounded-2xl p-4 shadow-sm space-y-3">
                                {cart.map(item => (
                                    <div key={item.id} className="flex justify-between items-center border-b border-slate-50 pb-3 last:border-0 last:pb-0">
                                        <div className="flex gap-3 items-center">
                                            <div className="text-2xl bg-slate-50 p-2 rounded-lg">{item.image_url}</div>
                                            <div>
                                                <h4 className="font-bold text-xs">{item.trade_name}</h4>
                                                <p className="text-emerald-600 font-black text-xs">{item.price_yer} ر.ي</p>
                                            </div>
                                        </div>
                                        <div className="flex items-center gap-2 bg-slate-50 rounded-lg p-1">
                                            <button onClick={()=>setCart(p=>p.map(i=>i.id===item.id?{...i,qty:Math.max(1, i.qty-1)}:i))} className="w-6 h-6 flex items-center justify-center bg-white rounded shadow-sm text-sm font-bold">-</button>
                                            <span className="text-xs font-bold w-4 text-center">{item.qty}</span>
                                            <button onClick={()=>setCart(p=>p.map(i=>i.id===item.id?{...i,qty:i.qty+1}:i))} className="w-6 h-6 flex items-center justify-center bg-white rounded shadow-sm text-sm font-bold">+</button>
                                        </div>
                                    </div>
                                ))}
                            </div>

                            <div className="bg-white rounded-2xl p-4 shadow-sm space-y-3">
                                <h3 className="font-bold text-xs text-slate-500 mb-2">بيانات التوصيل</h3>
                                <input type="text" value={address} onChange={e=>setAddress(e.target.value)} placeholder="عنوان التوصيل (المحافظة - الشارع)" className="w-full bg-slate-50 p-3 rounded-xl text-xs outline-none focus:border-emerald-500 border border-transparent" />
                                <input type="tel" value={phone} onChange={e=>setPhone(e.target.value)} placeholder="رقم الجوال للتواصل" className="w-full bg-slate-50 p-3 rounded-xl text-xs outline-none focus:border-emerald-500 border border-transparent" />
                            </div>

                            <div className="bg-white rounded-2xl p-4 shadow-sm space-y-2 text-xs">
                                <div className="flex justify-between text-slate-500"><span>المجموع:</span> <span>{total} ر.ي</span></div>
                                <div className="flex justify-between text-slate-500"><span>رسوم التوصيل:</span> <span>{deliveryFee} ر.ي</span></div>
                                <div className="flex justify-between font-black text-sm pt-2 border-t"><span>الإجمالي:</span> <span className="text-emerald-600">{total + deliveryFee} ر.ي</span></div>
                            </div>

                            <button onClick={handleCheckout} className="w-full bg-emerald-600 text-white font-black py-4 rounded-2xl shadow-lg hover:bg-emerald-700 active:scale-95 transition-all text-sm">
                                تأكيد الطلب والدفع عند الاستلام
                            </button>
                        </div>
                    )}
                </div>
            );
        }

        // --- Prescription Upload View ---
        function PrescriptionView({ user, db, notify, setActiveTab }) {
            const [image, setImage] = useState(null);
            const [loading, setLoading] = useState(false);

            const handleFile = (e) => {
                const file = e.target.files[0];
                if (file) {
                    // Convert to base64 for simulation (Canvas environment safety)
                    const reader = new FileReader();
                    reader.onloadend = () => setImage(reader.result);
                    reader.readAsDataURL(file);
                }
            };

            const uploadPrescription = async () => {
                if (!image) return;
                setLoading(true);
                try {
                    await db('orders').add({
                        userId: user.uid,
                        type: 'prescription',
                        imageUrl: image,
                        status: 'new',
                        timestamp: Date.now(),
                        customerName: "عميل",
                        phone: "777xxxxxx",
                        address: "سيتم تحديده لاحقاً"
                    });
                    notify('تم رفع الروشتة بنجاح! سيتم تسعيرها قريباً.');
                    setActiveTab('orders');
                } catch(e) {
                    notify('فشل الرفع', 'error');
                }
                setLoading(false);
            };

            return (
                <div className="space-y-4">
                    <h2 className="font-black text-lg border-b pb-2">رفع وصفة طبية</h2>
                    <div className="bg-white p-6 rounded-2xl shadow-sm border-2 border-dashed border-emerald-200 text-center relative">
                        <input type="file" accept="image/*" onChange={handleFile} className="absolute inset-0 w-full h-full opacity-0 cursor-pointer z-10" />
                        <div className="text-4xl mb-2">📸</div>
                        <h3 className="font-bold text-sm mb-1">التقط أو اختر صورة الروشتة</h3>
                        <p className="text-[10px] text-slate-400">سيلتزم الصيدلي بتوفير الأدوية المكتوبة بدقة</p>
                    </div>

                    {image && (
                        <div className="bg-white p-2 rounded-2xl shadow-sm">
                            <img src={image} alt="Preview" className="w-full h-48 object-cover rounded-xl" />
                            <button onClick={uploadPrescription} disabled={loading} className="w-full mt-3 bg-slate-900 text-white font-bold py-3 rounded-xl text-xs">
                                {loading ? 'جاري الإرسال...' : 'إرسال للصيدلية للمراجعة'}
                            </button>
                        </div>
                    )}
                </div>
            );
        }

        // --- Orders List View ---
        function OrdersView({ orders }) {
            const getStatusAr = (st) => {
                const s = { new: 'جديد بانتظار المراجعة', ready: 'قيد التجهيز', delivering: 'مع المندوب (في الطريق)', completed: 'مكتمل ✅', cancelled: 'ملغي ❌' };
                return s[st] || st;
            };
            const getStatusColor = (st) => {
                const s = { new: 'bg-blue-100 text-blue-700', ready: 'bg-amber-100 text-amber-700', delivering: 'bg-purple-100 text-purple-700', completed: 'bg-emerald-100 text-emerald-700' };
                return s[st] || 'bg-slate-100 text-slate-700';
            };

            return (
                <div className="space-y-4">
                    <h2 className="font-black text-lg border-b pb-2">طلباتي السابقة والحالية</h2>
                    {orders.length === 0 ? <p className="text-center text-xs text-slate-400 mt-10">لا توجد طلبات سابقة</p> : null}
                    
                    {orders.map(order => (
                        <div key={order.id} className="bg-white p-4 rounded-2xl shadow-sm border border-slate-100 space-y-3">
                            <div className="flex justify-between items-center border-b border-slate-50 pb-2">
                                <span className="text-[10px] text-slate-400 font-mono">#{order.id.slice(0,8)}</span>
                                <span className={`text-[9px] font-bold px-2 py-1 rounded-full ${getStatusColor(order.status)}`}>{getStatusAr(order.status)}</span>
                            </div>
                            
                            {order.type === 'prescription' ? (
                                <div className="flex gap-3">
                                    <img src={order.imageUrl} className="w-16 h-16 rounded-lg object-cover" />
                                    <div>
                                        <p className="font-bold text-xs text-slate-800">وصفة طبية مرفوعة</p>
                                        <p className="text-[10px] text-slate-500 mt-1">
                                            {order.total ? `تم التسعير: ${order.total} ر.ي` : 'بانتظار مراجعة الصيدلي للتسعير'}
                                        </p>
                                    </div>
                                </div>
                            ) : (
                                <div>
                                    <div className="flex flex-wrap gap-1 mb-2">
                                        {order.items.map((i, idx) => (
                                            <span key={idx} className="bg-slate-50 text-slate-600 text-[10px] px-2 py-0.5 rounded border border-slate-100">
                                                {i.qty}x {i.trade_name}
                                            </span>
                                        ))}
                                    </div>
                                    <div className="flex justify-between font-black text-sm text-slate-800 pt-1">
                                        <span>الإجمالي:</span>
                                        <span className="text-emerald-600">{order.total} ر.ي</span>
                                    </div>
                                </div>
                            )}

                            {order.status === 'delivering' && (
                                <div className="bg-teal-50 border border-teal-100 p-2 rounded-lg text-center animate-pulse">
                                    <p className="text-teal-800 text-[10px] font-bold">🛵 المندوب في الطريق إليك الآن!</p>
                                </div>
                            )}
                        </div>
                    ))}
                </div>
            );
        }

        // ==========================================
        // 2. ADMIN APP (Dashboard Interface)
        // ==========================================
        function AdminApp({ medicines, orders, notify, db }) {
            const [tab, setTab] = useState('orders');

            const updateOrderStatus = async (orderId, newStatus) => {
                try {
                    await db('orders').doc(orderId).update({ status: newStatus });
                    notify('تم تحديث حالة الطلب');
                } catch (e) {
                    notify('حدث خطأ', 'error');
                }
            };

            const addMedicine = async (e) => {
                e.preventDefault();
                const form = e.target;
                const newMed = {
                    trade_name: form.name.value,
                    scientific_name: form.sci.value,
                    category: form.cat.value,
                    price_yer: Number(form.price.value),
                    stock: Number(form.stock.value),
                    image_url: '💊', // Default
                    timestamp: Date.now()
                };
                try {
                    await db('medicines').add(newMed);
                    notify('تم إضافة الصنف بنجاح');
                    form.reset();
                } catch(err) {
                    notify('خطأ في الإضافة', 'error');
                }
            };

            return (
                <div className="pb-10">
                    <header className="bg-slate-900 text-white p-4 sticky top-0 z-30">
                        <h1 className="font-black text-lg">لوحة الإدارة ⚙️</h1>
                        <div className="flex gap-4 mt-3">
                            <button onClick={()=>setTab('orders')} className={`text-xs pb-1 font-bold ${tab==='orders'?'border-b-2 border-emerald-500 text-emerald-400':'text-slate-400'}`}>إدارة الطلبات ({orders.filter(o=>o.status==='new').length})</button>
                            <button onClick={()=>setTab('meds')} className={`text-xs pb-1 font-bold ${tab==='meds'?'border-b-2 border-emerald-500 text-emerald-400':'text-slate-400'}`}>المخزن والأدوية</button>
                        </div>
                    </header>

                    <div className="p-4">
                        {tab === 'orders' && (
                            <div className="space-y-4">
                                {orders.map(o => (
                                    <div key={o.id} className="bg-white p-4 rounded-xl shadow-sm border border-slate-200">
                                        <div className="flex justify-between items-start mb-3 border-b pb-2">
                                            <div>
                                                <span className="text-[10px] bg-slate-100 px-2 py-0.5 rounded text-slate-500 font-mono">#{o.id}</span>
                                                <p className="font-bold text-xs mt-1">{o.customerName} - {o.phone}</p>
                                                <p className="text-[10px] text-slate-500">📍 {o.address}</p>
                                            </div>
                                            <span className="font-black text-emerald-700 bg-emerald-50 px-2 py-1 rounded text-sm">{o.total ? `${o.total} ر.ي` : '?'}</span>
                                        </div>

                                        {o.type === 'prescription' && (
                                            <div className="mb-3">
                                                <p className="text-xs font-bold text-amber-600 mb-1">وصفة طبية (يرجى المراجعة والتسعير):</p>
                                                <img src={o.imageUrl} className="w-full h-32 object-contain bg-slate-50 border rounded-lg" />
                                            </div>
                                        )}

                                        {o.type === 'standard' && (
                                            <div className="text-[10px] text-slate-600 mb-3 bg-slate-50 p-2 rounded">
                                                {o.items.map(i=>`${i.qty}x ${i.trade_name}`).join(' ، ')}
                                            </div>
                                        )}

                                        <div className="flex gap-2">
                                            <select 
                                                value={o.status} 
                                                onChange={(e)=>updateOrderStatus(o.id, e.target.value)}
                                                className="bg-slate-100 border border-slate-200 text-xs p-2 rounded-lg flex-1 outline-none font-bold"
                                            >
                                                <option value="new">جديد (بانتظار المراجعة)</option>
                                                <option value="ready">جاهز (إسناد للمندوب)</option>
                                                <option value="delivering">جاري التوصيل</option>
                                                <option value="completed">مكتمل ✅</option>
                                                <option value="cancelled">إلغاء ❌</option>
                                            </select>
                                        </div>
                                    </div>
                                ))}
                            </div>
                        )}

                        {tab === 'meds' && (
                            <div className="space-y-6">
                                <form onSubmit={addMedicine} className="bg-white p-4 rounded-xl shadow-sm border border-slate-200 space-y-3">
                                    <h3 className="font-black text-sm border-b pb-2">إضافة صنف جديد</h3>
                                    <input required name="name" type="text" placeholder="الاسم التجاري" className="w-full bg-slate-50 border p-2 rounded text-xs outline-none" />
                                    <input required name="sci" type="text" placeholder="الاسم العلمي" className="w-full bg-slate-50 border p-2 rounded text-xs outline-none" />
                                    <div className="grid grid-cols-2 gap-2">
                                        <input required name="cat" type="text" placeholder="التصنيف (مثال: مسكنات)" className="bg-slate-50 border p-2 rounded text-xs outline-none" />
                                        <input required name="price" type="number" placeholder="السعر (ر.ي)" className="bg-slate-50 border p-2 rounded text-xs outline-none" />
                                    </div>
                                    <input required name="stock" type="number" placeholder="الكمية المتوفرة" className="w-full bg-slate-50 border p-2 rounded text-xs outline-none" />
                                    <button type="submit" className="w-full bg-slate-900 text-white font-bold py-2 rounded text-xs">حفظ الصنف</button>
                                </form>

                                <div className="bg-white p-4 rounded-xl shadow-sm border border-slate-200">
                                    <h3 className="font-black text-sm border-b pb-2 mb-3">الأصناف الحالية ({medicines.length})</h3>
                                    <div className="space-y-2">
                                        {medicines.map(m => (
                                            <div key={m.id} className="flex justify-between items-center text-xs border-b pb-2">
                                                <div>
                                                    <span className="font-bold">{m.trade_name}</span>
                                                    <p className="text-[10px] text-slate-500">المخزون: {m.stock} | {m.category}</p>
                                                </div>
                                                <span className="font-black text-emerald-600">{m.price_yer} ر.ي</span>
                                            </div>
                                        ))}
                                    </div>
                                </div>
                            </div>
                        )}
                    </div>
                </div>
            );
        }

        // ==========================================
        // 3. DELIVERY APP (Driver Interface)
        // ==========================================
        function DeliveryApp({ orders, notify, db }) {
            const updateOrderStatus = async (orderId, newStatus) => {
                try {
                    await db('orders').doc(orderId).update({ status: newStatus });
                    notify(newStatus === 'completed' ? 'تم تسليم الطلب بنجاح! أحسنت 👏' : 'تم استلام الطلب وبدء التوصيل');
                } catch (e) {
                    notify('حدث خطأ', 'error');
                }
            };

            return (
                <div className="h-full flex flex-col bg-slate-100">
                    <header className="bg-teal-700 text-white p-4 sticky top-0 z-30 flex justify-between items-center shadow-md">
                        <div>
                            <h1 className="font-black text-lg leading-none">تطبيق المندوب 🛵</h1>
                            <span className="text-[10px] text-teal-200">مستعد لاستقبال الطلبات</span>
                        </div>
                        <div className="w-3 h-3 bg-green-400 rounded-full shadow-[0_0_8px_rgba(74,222,128,0.8)] animate-pulse"></div>
                    </header>

                    <div className="p-4 space-y-4">
                        {orders.length === 0 && (
                            <div className="text-center py-20 opacity-50">
                                <div className="text-6xl mb-4">☕</div>
                                <p className="font-bold text-sm">لا توجد طلبات جاهزة للتوصيل حالياً.</p>
                            </div>
                        )}

                        {orders.map(o => (
                            <div key={o.id} className="bg-white rounded-2xl shadow-lg border border-slate-200 overflow-hidden">
                                {/* Simulated Google Map Section */}
                                <div className="h-32 bg-slate-200 relative overflow-hidden flex items-center justify-center">
                                    {/* Mock Map Background Grid */}
                                    <div className="absolute inset-0" style={{backgroundImage: 'linear-gradient(#cbd5e1 1px, transparent 1px), linear-gradient(90deg, #cbd5e1 1px, transparent 1px)', backgroundSize: '20px 20px', opacity: 0.5}}></div>
                                    <div className="relative z-10 flex flex-col items-center">
                                        <span className="text-3xl drop-shadow-md">📍</span>
                                        <span className="bg-white text-slate-800 text-[9px] font-bold px-2 py-1 rounded shadow mt-1">
                                            {o.address}
                                        </span>
                                    </div>
                                </div>

                                <div className="p-4">
                                    <div className="flex justify-between items-start mb-3">
                                        <div>
                                            <h3 className="font-black text-sm text-slate-800">{o.customerName}</h3>
                                            <p className="text-xs text-slate-600 font-mono mt-1">📞 {o.phone}</p>
                                        </div>
                                        <div className="text-left">
                                            <p className="text-[10px] text-slate-400 font-bold">المطلوب تحصيله</p>
                                            <p className="font-black text-lg text-emerald-600 leading-none mt-1">{o.total} <span className="text-[10px]">ر.ي</span></p>
                                        </div>
                                    </div>

                                    {o.status === 'ready' ? (
                                        <button 
                                            onClick={() => updateOrderStatus(o.id, 'delivering')}
                                            className="w-full bg-slate-900 text-white font-bold py-3 rounded-xl text-sm mt-2 shadow-md active:scale-95 transition-transform"
                                        >
                                            استلام الطلب والتوجه للعميل 🚀
                                        </button>
                                    ) : (
                                        <div className="flex gap-2 mt-2">
                                            <button 
                                                onClick={() => window.open(`tel:${o.phone}`)}
                                                className="flex-1 bg-blue-100 text-blue-700 font-bold py-3 rounded-xl text-xs flex justify-center items-center gap-1 active:bg-blue-200"
                                            >
                                                📞 اتصال
                                            </button>
                                            <button 
                                                onClick={() => updateOrderStatus(o.id, 'completed')}
                                                className="flex-[2] bg-emerald-600 text-white font-bold py-3 rounded-xl text-xs shadow-md active:scale-95 transition-transform"
                                            >
                                                ✅ تم التسليم والتحصيل
                                            </button>
                                        </div>
                                    )}
                                </div>
                            </div>
                        ))}
                    </div>
                </div>
            );
        }

        // --- Default Data ---
        const DEFAULT_MEDICINES = [
            { id: "med1", trade_name: "Panadol Advance", scientific_name: "Paracetamol 500mg", category: "مسكنات", price_yer: 1500, stock: 100, image_url: "💊" },
            { id: "med2", trade_name: "Amoxil 500mg", scientific_name: "Amoxicillin", category: "مضادات حيوية", price_yer: 2500, stock: 50, image_url: "💊" },
            { id: "med3", trade_name: "1,2,3 Cold & Flu", scientific_name: "Pseudoephedrine + Paracetamol", category: "نزلات البرد", price_yer: 1200, stock: 80, image_url: "🍵" },
        ];

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
