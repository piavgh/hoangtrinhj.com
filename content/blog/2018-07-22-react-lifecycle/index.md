---
author: Hoang Trinh
date: 2018-07-22 04:41:29+00:00
slug: react-lifecycle
title: Bàn một chút về React Lifecycle
template: post
thumbnail: '../thumbnails/react.png'
cover: './preview.jpg'
categories:
  - React.js
  - Popular
tags:
  - react.js
---

![react lifecycle](https://lecoder.io/wp-content/uploads/2018/07/react-lifecycle-1024x520.png)

_Bạn có thể check diagram này trên [http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)_

## Summary

Từ phiên bản 16.3 trở đi, React đã có lifecycle mới tuy nhiên vẫn hỗ trợ life cycle cũ.

Từ phiên bản 17 thì lifecycle cũ không được hỗ trợ nữa nên ở bài này chỉ để cập đến lifecyle hook mới của React.

Ở lifecycle cũ phát sinh nhiều vấn đề nhất là naming của lifecycle hook, gây hiểu nhầm!

Người ta thường dùng constructor, componentWillMount, comonentWillReceiveProps để init state dựa trên props, Vì vậy nhiều khi gây duplicate code. Đó là lý do mà getDerivedStateFromProps sinh ra để xoá sự khó hiểu và lằng nhằng đó đi.

Hoặc bạn muốn get thông tin nào đó của các ref trước khi render chẳng hạn, và sử dụng nó sau khi nó sau khi render lại.

Đó là lý do mà getSnapshotBeforeUpdate ra đời để thay thế componentWillUpdate, và componentDidUpdate có thêm một param
mới là snapshoot.

Tóm lại lifecycle mới sẽ có dạng tóm tắt như sau:

    Mounting: constructor → getDerivedStateFromProps → render → componentDidMount




    Updating: getDerivedStateFromProps → shouldComponentUpdate → render → getSnapshootBeforeUpdate → componentDidUpdate




    Unmouting: componentWillUnmount

## Nội dung

### Mounting(Component mới được khởi tạo)

☆ contructor(props): void

- Ở ES6 thì thường dùng để tạo init state, bind context(this) cho các function, event handling, createRef().

Ví dụ:

```jsx
class DemoComponent extends React.Component {
  constructor(props) {
    super(props)
    this.state = {}
    this.handleClick = this.handleClick.bind(this)
  }

  handleClick() {
    this.setState({})
  }

  render() {
    return <button onClick={this.handleClick}>Click</button>
  }
}
```

☆ static getDerivedStateFromProps(props, state): object

- Trước đây, với lifecycle cũ, người ta thường tính toán state thông qua props trong hàm constructor
  Và khi components update props thì dùng kèm với componentWillReceiveProps để set lại state.
  Hook này sinh ra để thay thế việc này.
- return về một object chính là state.
  Ví dụ với lifecycle cũ:

```jsx
    class DemoComponent extends React.Component {
        constructor(props) {
            super(props);
            this.state = {
                fullName: `${props.lastName} ${props.firstName}`,
            };
        }

        componentWillReceiveProps(nextProps, state) {
            this.setState({ fullName: `${nextProps.lastName} ${nextProps.firstName}` });
        }
        ...
    }
```

hoặc

```jsx
class DemoComponent extends React.Component {
  componentWillMount() {
    this.setState({
      fullName: `${this.props.lastName} ${this.props.firstName}`,
    })
  }

  componentWillReceiveProps(nextProps, state) {
    this.setState({ fullName: `${nextProps.lastName} ${nextProps.firstName}` })
  }
}
```

=> Khá là cực đúng không bạn?

Hãy chuyển sang lifecycle mới, nó sẽ trông như thế nào nhỉ?. Như này này:

```jsx
class DemoComponent extends React.Component {
  state = {}

  static getDerivedStateFromProps(props, state) {
    return {
      fullName: `${props.lastName} ${props.firstName}`,
    }
  }
}
```

☆ render(): ReactNode

☆ componentDidMount(prevProps, prevState): void

- Lúc này đã render lần đầu, lúc này thích hợp để tương tác với Tree Node

### Updating(Có cái gì đó thay đổi)

☆ static getDerivedStateFromProps(nextProps, state): object

- Khi props hoặc state thay đổi thì hook này được gọi(Từ phiên bản 16.4 trở đi thì state hay props thay đổi thì hook này được gọi)

☆ shouldComponentUpdate(nextProps, nextState): boolean

- Khi props hoặc state thay đổi thì hook này được gọi
- Với PureComponent thì dev không thể định nghĩa lại. Mặc định là shallow compare.
- Với Component thì dev có thể định nghĩa lại để chống render không cần thiết. Mặc định là reference compare.
- Nếu xác định component này chỉ render 1 lần thì return false, chống render(Đối với Component).
- Nếu return false thì sẽ không tới hook re-render tiếp theo.

Note: Để thấy được sự khác nhau giữa PureComponent và Component, hãy vào link dưới:
https://lecoder.io/component_types
https://lecoder.io/component_vs_purecomponent

☆ render(): ReactNode

☆ getSnapshootBeforeUpdate(prevProps, prevState): object

- Thường thì get một số thông tin của props, state, hoặc ref trước khi re-render, và sử dụng nó sau khi render.

☆ componentDidUpdate(prevProps, prevState, snapshoot): void

- Lúc này đã re-render, thích hợp để tương tác với Tree Node.
- Với lifecycle mới thì có thêm param snapshoot, snapshoot là output của getSnapshootBeforeUpdate hook

### Unmounting (Chuyển sang component khác)

☆ componentWillUnmount: void

- Thường thì chạy một function nào đó, ví dụ như clear interval, delete rác. Ít khi sử dụng.
- Không nên setState tại đây, Vì nó chết rồi không sống lại nữa.

## Một vài chia sẻ khác

- Hiện tại là phiên bản 16.4.1 Lifecycle hook khá nhiều vì phải hỗ trợ cả hai. Khuyến khích nên sử dụng lifecycle mới [https://reactjs.org/docs/react-component.html](https://reactjs.org/docs/react-component.html)

- Khuyến khích sử dụng coding standar của airbnb cho React [https://github.com/airbnb/javascript/tree/master/react](https://github.com/airbnb/javascript/tree/master/react)

- Khuyến khích sử dụng class ES7 thay thế cho ES6 để tránh việc bind this, init state, propTypes, defaultPropTypes, createRef()

_Nguồn tham khảo: https://github.com/nguyenvanhoang26041994/dev-experiences/blob/master/React/lifecycle_hook_
