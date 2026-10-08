import streamlit as st
import pandas as pd
from datetime import datetime

# --- CẤU HÌNH TRANG ---
st.set_page_config(
    page_title="Quản Lý Đặt Trà Sữa",
    page_icon="🧋",
    layout="wide"
)

# --- KHỞI TẠO DỮ LIỆU SẢN PHẨM & GIÁ ---
MENU_TRA_SUA = {
    "Trà sữa Truyền thống": 30000,
    "Trà sữa Ô long": 35000,
    "Trà sữa Trái cây": 35000,
    "Trà sữa Matcha": 40000,
    "Trà sữa Socola": 40000,
    "Trà Đào Cam Sả": 38000
}

PRICE_SIZE = {
    "Nhỏ (S)": 0,
    "Vừa (M)": 5000,
    "Lớn (L)": 10000
}

TOPPING_MENU = {
    "Trân châu đen": 5000,
    "Trân châu trắng": 7000,
    "Thạch trái cây": 5000,
    "Pudding trứng": 8000,
    "Kem cheese": 10000
}

SUGAR_LEVELS = ["100% (Chuẩn)", "70%", "50%", "30%", "0% (Không đường)"]
ICE_LEVELS = ["100% (Chuẩn)", "70%", "50%", "30%", "0% (Không đá)"]

# --- QUẢN LÝ STATE GIỎ HÀNG ---
if "cart" not in st.session_state:
    st.session_state.cart = []

if "order_placed" not in st.session_state:
    st.session_state.order_placed = False

def add_to_cart(item):
    st.session_state.cart.append(item)
    st.session_state.order_placed = False

def clear_cart():
    st.session_state.cart = []
    st.session_state.order_placed = False

# --- GIAO DIỆN CHÍNH ---
st.title("🧋 Hệ Thống Đặt Hàng & Tính Tiền Trà Sữa")
st.markdown("---")

col_left, col_right = st.columns([1.2, 1], gap="large")

# === CỘT TRÁI: NHẬP THÔNG TIN & CHỌN MÓN ===
with col_left:
    st.header("📋 Thông Tin Đơn Hàng")
    
    # 1. Thông tin khách hàng
    customer_name = st.text_input("Tên khách hàng:", placeholder="Nhập tên khách hàng...")
    
    st.subheader("🥤 Chọn Đồ Uống")
    
    # 2. Chọn trà sữa & size
    col_drink, col_size = st.columns(2)
    with col_drink:
        drink_name = st.selectbox("Loại trà sữa:", list(MENU_TRA_SUA.keys()))
    with col_size:
        size = st.selectbox("Kích thước (Size):", list(PRICE_SIZE.keys()))
        
    # 3. Chọn mức đường & đá
    col_sugar, col_ice = st.columns(2)
    with col_sugar:
        sugar = st.selectbox("Mức độ đường:", SUGAR_LEVELS)
    with col_ice:
        ice = st.selectbox("Mức độ đá:", ICE_LEVELS)
        
    # 4. Chọn Topping & Số lượng
    toppings = st.multiselect("Thêm Topping:", list(TOPPING_MENU.keys()))
    quantity = st.number_input("Số lượng:", min_value=1, value=1, step=1)
    
    # Tính giá tiền món hiện tại
    base_price = MENU_TRA_SUA[drink_name]
    size_price = PRICE_SIZE[size]
    topping_price = sum(TOPPING_MENU[t] for t in toppings)
    unit_price = base_price + size_price + topping_price
    item_total = unit_price * quantity
    
    st.markdown(f"**Đơn giá:** {unit_price:,.0f} VNĐ / ly")
    st.markdown(f"**Thành tiền món này:** `{item_total:,.0f} VNĐ`")
    
    # Nút thêm vào đơn hàng
    if st.button("➕ Thêm vào hóa đơn", type="primary", use_container_width=True):
        if not customer_name.strip():
            st.error("⚠️ Vui lòng nhập tên khách hàng trước khi thêm món!")
        else:
            item = {
                "drink": drink_name,
                "size": size,
                "sugar": sugar,
                "ice": ice,
                "toppings": ", ".join(toppings) if toppings else "Không",
                "quantity": quantity,
                "unit_price": unit_price,
                "total": item_total
            }
            add_to_cart(item)
            st.toast("✅ Đã thêm món vào hóa đơn!", icon="🧋")

# === CỘT PHẢI: XEM GIỎ HÀNG & HÓA ĐƠN ===
with col_right:
    st.header("🛒 Chi Tiết Hóa Đơn")
    
    if not st.session_state.cart:
        st.info("Chưa có món nào được chọn. Vui lòng chọn món ở cột bên trái.")
    else:
        # Hiển thị bảng các món đã chọn
        cart_data = []
        total_bill = 0
        
        for idx, item in enumerate(st.session_state.cart, 1):
            total_bill += item["total"]
            cart_data.append({
                "STT": idx,
                "Tên món": f"{item['drink']} ({item['size']})",
                "Tùy chọn": f"Đường: {item['sugar']} | Đá: {item['ice']} | Topping: {item['toppings']}",
                "SL": item["quantity"],
                "Tổng (VNĐ)": f"{item['total']:,.0f}"
            })
            
        df_cart = pd.DataFrame(cart_data)
        st.dataframe(df_cart.drop(columns=["Tùy chọn"]), use_container_width=True, hide_index=True)
        
        # Chi tiết chi tiết từng ly bên dưới
        with st.expander("🔍 Xem chi tiết topping & tùy chỉnh"):
            for i, item in enumerate(st.session_state.cart, 1):
                st.write(f"**{i}. {item['drink']} ({item['size']})** x{item['quantity']}")
                st.caption(f"- Đường: {item['sugar']} | Đá: {item['ice']}")
                st.caption(f"- Topping: {item['toppings']}")
                st.divider()

        st.subheader(f"💰 TỔNG TIỀN: :{st.write(f'### {total_bill:,.0f} VNĐ') if False else ''} {total_bill:,.0f} VNĐ")
        
        col_pay, col_clear = st.columns([2, 1])
        with col_pay:
            if st.button("💳 Thanh Toán & In Hóa Đơn", type="primary", use_container_width=True):
                st.session_state.order_placed = True
        with col_clear:
            if st.button("🗑️ Xóa đơn", use_container_width=True):
                clear_cart()
                st.rerun()

    # === HIỂN THỊ HÓA ĐƠN HOÀN CHỈNH KHI BẤM THANH TOÁN ===
    if st.session_state.order_placed and st.session_state.cart:
        st.markdown("---")
        st.success("🎉 Thanh toán thành công!")
        
        # Khung hóa đơn
        with st.container(border=True):
            st.markdown("<h3 style='text-align: center;'>🧋 HÓA ĐƠN BÁN HÀNG</h3>", unsafe_allow_html=True)
            st.write(f"**Khách hàng:** {customer_name}")
            st.write(f"**Thời gian:** {datetime.now().strftime('%d/%m/%Y %H:%M:%S')}")
            st.markdown("---")
            
            final_total = 0
            for idx, item in enumerate(st.session_state.cart, 1):
                final_total += item["total"]
                st.markdown(f"**{idx}. {item['drink']}** - Size {item['size']} x {item['quantity']}")
                st.text(f"   Đường: {item['sugar']} | Đá: {item['ice']}")
                st.text(f"   Topping: {item['toppings']}")
                st.markdown(f"   *Thành tiền:* **{item['total']:,.0f} VNĐ**")
                st.markdown("---")
                
            st.markdown(f"### 💵 Tổng thanh toán: `{final_total:,.0f} VNĐ`")
            st.markdown("<p style='text-align: center; color: gray;'>Cảm ơn quý khách và hẹn gặp lại!</p>", unsafe_allow_html=True)
