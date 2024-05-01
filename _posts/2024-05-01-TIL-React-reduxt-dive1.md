---
title: React redux 깊게 다뤄보기1
date: 2024-05-01 22:40:00 +09:00
categories: [TIL, React]
tags:
  [
    TIL,
    javascript,
    React,
    redux,
    redux-toolkit
  ]
image: ../assets/img/common/react-logo.png
---

# redux-toolkit 을 이용한 실습 1

- source coude git hub : [ykdman/Study/react-redux-dive](https://github.com/ykdman/Study/tree/main/react-redux-dive)

## 제공받은 코드에서 다중 Slice를 만들어 보기

- Header에 CartButton 컴포넌트를 누르면 Cart 컴포넌트가 보이게 하는 Slice
- 메인화면에서 상품 의 'Add to Cart' 를 누르면 Cart에 상품이 추가되게 하는 Slice

  - 추가된 상품이 이미 있을 경우, 누적증가, 누적감소가 수행되게 해야한다!

### 1. <Cart /> Visible Slice 만들기

우선 store 폴더를 만들어 그곳에서 다중 상태 관리가 가능한 store.js 파일과

Cart 컴포넌트를 toggle 해줄 전역 상태 Slice 가 필요했다.

Cart 컴포넌트 표시를 toggle 해줄 js → store \ cartUi.slice.js

전역 store 저장 js → store \ store.js 로 생성


- cartUi.Slice.js 파일은 이렇게 작성했다.
    
    ```js
    import { createSlice } from "@reduxjs/toolkit";
    
    const initialState = { cartIsVisible: false };
    
    const cartUiSlice = createSlice({
      name: "cartUi",
      initialState: initialState,
      reducers: {
        toggle(state) {
          state.cartIsVisible = !state.cartIsVisible;
        },
      },
    });
    
    export const cartUiActions = cartUiSlice.actions;
    export default cartUiSlice;
    
    ```
    
    - reducer 함수로 toggle 기능을 할 함수만 필요하기에 1개만 선언하였다.
        - toggle 함수는 cartUiSlice의 state에서 cartIsVisible 값만 toggle 하면 된다.
- 그 이후로 store.js 파일을 작성하였다.
    
    ```js
    import { configureStore } from "@reduxjs/toolkit";
    import cartUiSlice from "./cartUi.slice";
    
    const store = configureStore({
      reducer: { cartUi: cartUiSlice.reducer },
    });
    
    export default store;
    
    ```
    
    - 처음에 store의 reducer 의 속성으로 cartUi : cartUiSlice 만 써서 toggle 이 안되었다….
        - 이전에 redux를 배울때 작성 했었던 코드를 보니 .reducer 까지 작성해야 했다. ㅠㅠ
- Cart 컴포넌트는 App 상에 있기 때문에 cartUiSlice 의 값을 가져와야 했다.

```js
// App.js
import { useSelector } from "react-redux";
import Cart from "./components/Cart/Cart";
import Layout from "./components/Layout/Layout";
import Products from "./components/Shop/Products";

function App() {
  const showCart = useSelector((state) => state.cartUi.cartIsVisible);

  return (
    <Layout>
      {showCart && <Cart />}  {/*Cart Toggle*/}
      <Products />
    </Layout>
  );
}

export default App;

```

- 그리고 Layout 안에 MainHeader 컴포넌트 안에 CartButton 이 존재했다.
    - CartButton 컴포넌트를 누르면, Cart 컴포넌트가 화면에 떠야한다!
        
        ```js
        import { useDispatch } from "react-redux";
        import { cartUiActions } from "../../store/cartUi.slice";
        import classes from "./CartButton.module.css";
        
        const CartButton = (props) => {
          // cart is Visible Flag
          // const cartIsVisible = useSelector((state) => state.cartIsVisible);
          const disPatch = useDispatch();
        
          const toggleCartHandler = () => {
            disPatch(cartUiActions.toggle());
          };
        
          return (
            <button className={classes.button} onClick={toggleCartHandler}>
              <span>My Cart</span>
              <span className={classes.badge}>1</span>
            </button>
          );
        };
        
        export default CartButton;
        
        ```
        

## 2. Cart 컴포넌트에 물품 수량 추가 및 빼기 기능 작성

- 상품을 추가하면 Cart 컴포넌트에 누적 되게 하기 위해 작성해야할 코드
    - Cart.js
    - CartItem.js
    - Products.js
    - ProductItem.js
    - cart.slice.js
- cart.slice.js
    
    ```js
    import { createSlice } from "@reduxjs/toolkit";
    
    const initialState = {
      items: [],
      totalQty: 0,
      totalPrice: 0,
    };
    
    const cartSlice = createSlice({
      name: "cart",
      initialState,
      reducers: {
        addItemToCart(state, action) {
          // redux에는 action의 나머지 property 가 payload 에 들어있다.
          const newItem = action.payload;
          // 참조값 사용!
          const existingItem = state.items.find((item) => item.id === newItem.id);
          state.totalQty++;
          if (!existingItem) {
            // 최초로 item 추가
            state.items.push({
              id: newItem.id,
              price: newItem.price,
              quantity: 1,
              totalPrice: newItem.price,
              name: newItem.title,
            });
          } else {
            // 이미 있는 item을 추가
            // existingItem이 Object 참조값이기 때문에, existing itme의 속성을 수정하면, item의 요소에도 반영된다.
            existingItem.quantity = existingItem.quantity + 1;
            existingItem.totalPrice = existingItem.totalPrice + newItem.price;
          }
        },
        removeItemFromCart(state, action) {
          const { id } = action.payload;
          const existingItem = state.items.find((item) => item.id === id);
          state.totalQty--;
          if (existingItem.quantity === 1) {
            // 줄이는 item 의 수량이 1 이면 삭제
            state.items = state.items.filter((item) => item.id !== id);
          } else {
            // 줄이는 item의 수량이 1이상이면 감소
            existingItem.quantity = existingItem.quantity - 1;
            // 1개를 빼는 것이기 때문에 총 가격도 1개분의 감소가 일어나야함
            existingItem.totalPrice = existingItem.totalPrice - existingItem.price;
          }
        },
      },
    });
    
    export const cartActions = cartSlice.actions;
    export default cartSlice;
    
    ```
    
    - cartSlice 라는 slice를 만들었다.
    - cartSlice의 state 중 items 배열에 상품이 추가되고 삭제 된다.
        - 상품추가(갯수증가) : addItemToCart
        - 상품제거(갯수감소) : removeItemFromCart
    - 상품 추가 및 제거 함수를 작성하면서 아주 크게 간과했던게 있다.
        - existingItem 이라는 변수에 기존 state.items에서 찾은 요소를 할당 하는데 이게 `참조값` 이기 때문에, existingItem 객체의 내부 값을 수정하면, state.items 에도 그대로 반영된다는 것을 문득 깨달았다…
        - 기본기가 참으로 중요하다.
- Products.js
    
    ```js
    import ProductItem from "./ProductItem";
    import classes from "./Products.module.css";
    
    const DUMMY_ITEMS = [
      {
        id: "p1",
        price: 6,
        title: "my first Book",
        description: "firstBook i ever read",
      },
      {
        id: "p5",
        price: 7,
        title: "my seconde Book",
        description: "second Book i ever read",
      },
      {
        id: "p3",
        price: 8,
        title: "my Third Book",
        description: "third Book i ever read",
      },
      {
        id: "p4",
        price: 9,
        title: "my fourth Book",
        description: "fourth Book i ever read",
      },
      {
        id: "p5",
        price: 6,
        title: "my fifth Book",
        description: "fifth Book i ever read",
      },
    ];
    
    const Products = (props) => {
      return (
        <section className={classes.products}>
          <h2>Buy your favorite products</h2>
          <ul>
            {DUMMY_ITEMS.map((item) => {
              return (
                <ProductItem
                  id={item.id}
                  key={item.id}
                  title={item.title}
                  price={item.price}
                  description={item.description}
                />
              );
            })}
            {/* <ProductItem
              title="Test"
              price={6}
              description="This is a first product - amazing!"
            /> */}
          </ul>
        </section>
      );
    };
    
    export default Products;
    
    ```
    
    - 우선 더미 데이터를 만들어 map 함수로 각 요소 마다 ProductItem으로 렌더링 되게 작성하였다.
- ProductItem.js
    
    ```js
    import { useDispatch } from "react-redux";
    import Card from "../UI/Card";
    import classes from "./ProductItem.module.css";
    import { cartActions } from "../../store/cart.slice";
    
    const ProductItem = (props) => {
      const { id, title, price, description } = props;
      const item = {
        id,
        title,
        price,
      };
      const disPatch = useDispatch();
      const addtoCartHandler = () => {
        disPatch(
          cartActions.addItemToCart({
            id: item.id,
            price: item.price,
            title: item.title,
          })
        );
      };
      return (
        <li className={classes.item}>
          <Card>
            <header>
              <h3>{title}</h3>
              <div className={classes.price}>${price.toFixed(2)}</div>
            </header>
            <p>{description}</p>
            <div className={classes.actions}>
              <button onClick={addtoCartHandler}>Add to Cart</button>
            </div>
          </Card>
        </li>
      );
    };
    
    export default ProductItem;
    
    ```
    
    - ProductItem 은 ‘Add to Cart’ 버튼을 누르면 cart.state.items에 영향을 주어야 하게 만들었다.
    - cartSlice 의 actions 인 cartActions 를 import 하여 버튼을 누르면 상품이 추가되게 하였다.
- Cart.js
    
    ```js
    import { useSelector } from "react-redux";
    import Card from "../UI/Card";
    import classes from "./Cart.module.css";
    import CartItem from "./CartItem";
    
    const Cart = (props) => {
      const productItems = useSelector((state) => state.cart.items);
      return (
        <Card className={classes.cart}>
          <h2>Your Shopping Cart</h2>
          <ul>
            {productItems.map((product) => (
              <CartItem
                item={{
                  id: product.id,
                  title: product.name,
                  quantity: product.quantity,
                  total: product.quantity * product.price,
                  price: product.price,
                }}
              />
            ))}
            {/* <CartItem
              item={{ title: 'Test Item', quantity: 3, total: 18, price: 6 }}
            /> */}
          </ul>
        </Card>
      );
    };
    
    export default Cart;
    
    ```
    
    - Cart 는 실상 state.cart.items를 렌더링 하는 목적과, 그안에서 증가,감소를 위해 사용되었다.
    - Cart 안에서 CartItem을 렌더링 하며, 해당 item에서 버튼을 누르면 증가, 감소가 수행된다.
- CartItem.js
    
    ```js
    import { useDispatch } from "react-redux";
    import classes from "./CartItem.module.css";
    import { cartActions } from "../../store/cart.slice";
    
    const CartItem = (props) => {
      const { id, title, quantity, total, price } = props.item;
      const disPatch = useDispatch();
    
      const removeFromCartHandler = () => {
        disPatch(cartActions.removeItemFromCart({ id }));
      };
    
      const addToCartHandeler = () => {
        disPatch(cartActions.addItemToCart({ id, price, title }));
      };
    
      return (
        <li className={classes.item}>
          <header>
            <h3>{title}</h3>
            <div className={classes.price}>
              ${total.toFixed(2)}{" "}
              <span className={classes.itemprice}>(${price.toFixed(2)}/item)</span>
            </div>
          </header>
          <div className={classes.details}>
            <div className={classes.quantity}>
              x <span>{quantity}</span>
            </div>
            <div className={classes.actions}>
              <button onClick={removeFromCartHandler}>-</button>
              <button onClick={addToCartHandeler}>+</button>
            </div>
          </div>
        </li>
      );
    };
    
    export default CartItem;
    
    ```
    
    - CartItem도 ProductItem 과 비슷하게 카트에 추가, 제거가 가능한 코드를 작성하였다.