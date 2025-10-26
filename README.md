<script>
const DELIVERY_CHARGE = 100;
const PRODUCTS = [
  { id:1, title:'Chocolate Cupcake (1 pc)', cat:'cupcake', price:80, desc:'Rich chocolate cupcake with chocolate chips.' },
  { id:2, title:'Vanilla Cupcake (1 pc)', cat:'cupcake', price:70, desc:'Classic vanilla cupcake with buttercream.' },
  { id:3, title:'Red Velvet Cupcake (1 pc)', cat:'cupcake', price:80, desc:'Soft red velvet with cream cheese frosting.' },
  { id:4, title:'Oreo Cupcake (1 pc)', cat:'cupcake', price:60, desc:'Chocolate cupcake topped with Oreo crumbs.' },
  { id:5, title:'Strawberry Cupcake (1 pc)', cat:'cupcake', price:70, desc:'Vanilla cupcake with strawberry frosting and sprinkles.' },
  { id:6, title:'Mini Cookie (1 pc)', cat:'cookie', price:40, desc:'Buttery cookie with chocolate.' },
  { id:7, title:'Birthday Cake (Half kg)', cat:'cake', price:500, desc:'Custom message available.' }
];

let CART = {};
let CART_META = { address: '', location: '' };

const productsEl = document.getElementById('products');
const countEl = document.getElementById('count');
const cartBtn = document.getElementById('cartBtn');
const drawer = document.getElementById('drawer');
const cartItemsEl = document.getElementById('cartItems');
const cartTotalEl = document.getElementById('cartTotal');

function renderProducts(list){
  productsEl.innerHTML = '';
  countEl.textContent = list.length;
  list.forEach(p => {
    const card = document.createElement('div'); card.className='card';
    const img = document.createElement('div'); img.className='img'; img.textContent=p.title.split(' ')[0];
    const title = document.createElement('div'); title.textContent=p.title; title.style.fontWeight=700;
    const desc = document.createElement('div'); desc.className='muted'; desc.textContent = p.desc;
    const price = document.createElement('div'); price.className='price'; price.textContent = `₹${p.price}`;
    const qtyWrap = document.createElement('div');
    qtyWrap.style.display='flex'; qtyWrap.style.gap='8px'; qtyWrap.style.alignItems='center';
    const qtyLabel = document.createElement('div'); qtyLabel.className='muted'; qtyLabel.textContent='Qty';
    const qtyInput = document.createElement('input'); qtyInput.type='number'; qtyInput.min=1; qtyInput.value=1;
    qtyInput.style.width='64px'; qtyInput.style.padding='6px'; qtyInput.style.borderRadius='6px';
    qtyInput.style.background='transparent'; qtyInput.style.border='1px solid rgba(255,255,255,0.06)';
    qtyInput.style.color='#fff';
    qtyWrap.append(qtyLabel, qtyInput);

    const actions = document.createElement('div'); actions.className='actions';
    const add = document.createElement('button'); add.className='btn add'; add.textContent='Add';
    add.onclick = ()=>addToCartWithQty(p.id, parseInt(qtyInput.value||1,10));
    const buy = document.createElement('button'); buy.className='btn view'; buy.textContent='Buy Now';
    buy.onclick = ()=>{ addToCartWithQty(p.id, parseInt(qtyInput.value||1,10)); openCart(); };
    actions.append(add, buy);

    card.append(img, title, desc, price, qtyWrap, actions);
    productsEl.appendChild(card);
  });
}

function addToCartWithQty(id, qty){
  if(!qty || qty < 1) qty = 1;
  CART[id] = CART[id] ? CART[id] + qty : qty;
  updateCartUI();
  alert('Added to cart!');
}

function updateCartUI(){
  const cartCount = Object.values(CART).reduce((s,v)=>s+v,0);
  document.getElementById('cartCount').textContent=cartCount;
  renderCartItems();
}

function renderCartItems(){
  cartItemsEl.innerHTML = '';
  let total = 0;
  const keys = Object.keys(CART);
  if(keys.length===0){ cartItemsEl.innerHTML = '<div class="muted">Your cart is empty.</div>'; cartTotalEl.textContent = '₹0'; return }
  keys.forEach(k=>{
    const p = PRODUCTS.find(x=>x.id==k);
    const qty = CART[k];
    const line = document.createElement('div'); line.className='cart-item';
    const left = document.createElement('div'); left.innerHTML = `<strong>${p.title}</strong><div class='muted'>₹${p.price} × ${qty}</div>`;
    const right = document.createElement('div');
    const lineTotal = p.price * qty;
    total += lineTotal;
    right.innerHTML = `<div>₹${lineTotal}</div>
      <div style='margin-top:6px'>
        <button class='btn view' onclick='decrease(${p.id})'>-</button>
        <button class='btn add' onclick='increase(${p.id})'>+</button>
      </div>`;
    line.append(left, right);
    cartItemsEl.appendChild(line);
  });

  const grand = total + DELIVERY_CHARGE;
  cartTotalEl.textContent = `₹${grand} (incl. delivery ₹${DELIVERY_CHARGE})`;

  // Address section
  const addrWrap = document.createElement('div'); addrWrap.style.marginTop='12px';
  addrWrap.innerHTML = `
    <label class='muted'>Delivery Address (required)</label>
    <textarea id='addressInput' rows='2' style='width:100%;padding:8px;border-radius:8px;border:1px solid rgba(255,255,255,0.06);background:transparent;color:#fff'>${CART_META.address || ''}</textarea>
    <div style='display:flex;gap:8px;margin-top:8px'>
      <button id='useLoc' class='btn view'>Use My Location</button>
      <button id='saveAddr' class='btn add'>Save Address</button>
    </div>
  `;
  cartItemsEl.appendChild(addrWrap);

  setTimeout(()=>{
    const useLoc = document.getElementById('useLoc');
    const saveAddr = document.getElementById('saveAddr');
    const addressInput = document.getElementById('addressInput');

    useLoc.onclick = ()=>{
      if(navigator.geolocation){
        navigator.geolocation.getCurrentPosition(pos=>{
          const lat = pos.coords.latitude.toFixed(6);
          const lon = pos.coords.longitude.toFixed(6);
          const text = `Lat: ${lat}, Lon: ${lon}`;
          addressInput.value = text;
          CART_META.address = text;
          alert('Location detected successfully!');
        }, ()=> alert('Please allow location permission.'));
      } else alert('Geolocation not supported on this device.');
    };

    saveAddr.onclick = ()=>{
      CART_META.address = addressInput.value.trim();
      if(!CART_META.address) return alert('Address cannot be empty');
      alert('Address saved successfully!');
    };
  },50);
}

function increase(id){ CART[id] = (CART[id]||0) + 1; updateCartUI(); }
function decrease(id){ if(!CART[id]) return; CART[id]--; if(CART[id] <= 0) delete CART[id]; updateCartUI(); }

function openCart(){ drawer.style.display = 'block'; renderCartItems(); }
cartBtn.onclick = ()=>{ openCart(); };

document.getElementById('checkout').addEventListener('click', async ()=>{
  const keys = Object.keys(CART);
  if(keys.length===0){ alert('Your cart is empty!'); return }
  if(!CART_META.address || CART_META.address.trim().length < 5){ alert('Delivery address is required.'); return }
  const total = keys.reduce((s,k)=> s + CART[k] * (PRODUCTS.find(p=>p.id==k).price),0);
  const orderData = {
    items: keys.map(k=>({title: PRODUCTS.find(p=>p.id==k).title, qty: CART[k]})),
    total: total + DELIVERY_CHARGE,
    address: CART_META.address,
    time: new Date().toLocaleString()
  };

  // send order to your backend (replace with your real endpoint)
  try {
    const res = await fetch('https://webhook.site/your-unique-url', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(orderData)
    });
    alert('Order placed! Delivery in 30–45 mins.\nCheck your host dashboard.');
  } catch(e){
    alert('Order placed locally (network error).');
  }

  CART = {}; CART_META.address = '';
  updateCartUI(); drawer.style.display='none';
});

document.getElementById('clearCart').addEventListener('click', ()=>{ CART = {}; CART_META.address=''; updateCartUI(); });
renderProducts(PRODUCTS);
</script>
