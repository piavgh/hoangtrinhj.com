---
author: hoangth
date: 2018-07-21 11:30:48+00:00
slug: toi-uu-performance-cho-react
title: Tối ưu performance cho React
thumbnail: '../thumbnails/react.png'
template: post
categories:
  - React.js
tags:
  - react.js
---

Bạn mới bắt đầu với React? Bạn đang băn khoăn tại sao càng code nhiều thì website chạy càng chậm? Bạn không biết làm sao để khi test trên Google PageSpeed Insights đạt được kết quả tốt hơn?

Mình cũng đã từng có những băn khoăn đó.

Hôm nay mình sẽ hướng dẫn cho các bạn một vài thủ thuật để làm cho website React của bạn chạy nhanh hơn, mượt mà hơn, giữ chân người dùng ở lại lâu hơn, blabla…

## Summary

Lúc còn mới tập tành code React file bundle của mình thường lên tới từ 1Mb đến 6Mb. Chạy lần đầu trên product chậm rì
chậm rịt. Sau gần 1 năm dùng React thì bản thân cũng rút ra một cố cách dùng để cải thiện performance.

Bài viết này sẽ xoanh quanh vấn đề làm sao để app chạy nhanh nhất ngay cả lần đầu vào trang web.
Các kỹ thuật mình nói trong bài viết sẽ bao gồm:
☆ Code splitting, đây là kỹ thuật trọng điểm trong bài này.
☆ Production build(with webpack)
☆ Nén file với gzip
☆ Server-side-rendering
☆ CDN(Content Delivery Network)
☆ Web worker

Và tất nhiên mình không giới thiệu một cách detail. Mình chỉ nói để các bạn hiểu những kĩ thuật đó là gì, tại sao
phải sử dụng nó, sử dụng khi nào. Các bạn có thể tìm thông tin chi tiết trên Google rất nhiều.

## Nội dung

### Code splitting

Đây là nòng cốt của bất kỳ bạn frontend developer nào cũng cần phải biết. Câu chuyện là hồi xưa mình thường dùng webpack để bundle ra một file duy nhất là bundle.js rồi import nó vào trang index.html qua thẻ < script>.
Chuyện cũng không có gì cho đến một thời gian sau code mình càng ngày càng lớn khiến file bundle càng ngày càng nặng.
Bạn đầu cũng 200kb, rồi 1Mb, rồi lên tới 2Mb. Chắc nhiều bạn mới cũng gặp cái bí này. Và rồi thông qua techtalk.vn
(hay viblo gì đó không nhớ) mình biết tới code spliting.

Ví dụ, bạn có trang chủ: localhost:3000/ chứa link tới các trang /login, /about, /promotion

Tương ứng với mỗi route đó là các React.Component sau:
– HomePage cho trang chủ /
– AboutUs cho /about
– Promotion cho /promotion

Okay!, giờ code cho mấy cái route này nào.

####

Step 1: (App.js)

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import { BrowserRouter as Router, Route, Link } from 'react-router-dom'

import HomePage from '../path/to/components/HomePage'
import AboutUs from '../path/to/components/AboutUs'
import Promotion from '../path/to/components/Promotion'

const App = () => (
  <Router>
    <div>
      <Link to="/">Trang chủ</Link>
      <Link to="/about">Về chúng tôi</Link>
      <Link to="/promotion">Nhận khuyến mãi ngay</Link>
      <Route exact path="/" component={HomePage} />
      <Route path="/about" component={AboutUs} />
      <Route path="promotion" component={Promotion} />
    </div>
  </Router>
)

ReactDOM.render(<App />, document.getElementById('app'))
```

Xong, chạy app lên bạn sễ thấy file bundle của bạn nặng 2Mb(giả sử). Trong có soure code của HomePage tầm 300kB,
AboutUs tầm 200Kb, Promotion tầm 500Kb, mấy cái viện như React, Lodash tổng cộng… tầm 1Mb đi chẳng hạn.

Câu hỏi đặt ra là: Ủa tại sao tôi vào Trang Chủ mà bắt tôi tải soure của mấy trang Khuyến Mãi, rồi Thông tin công ty
làm gì vậy hè?. Hoặc tôi vào localhost:3000/promotion thì chỉ càn tải source của Trang khuyến mãi thôi chứ tải Trang Chủ
làm gì?.

Okay!, tiếp nào. Thì đây chính là code sau khi sử dụng code splitting(Sử dụng một thư viên là react-loadable)
https://github.com/jamiebuilds/react-loadable

####

Step 2:(App.js)

```jsx
import React from 'react'
import Loadable from 'react-loadable'
import ReactDOM from 'react-dom'
import { BrowserRouter as Router, Route, Link } from 'react-router-dom'

const Spinner = () => <span className="spinner" /> // Một cái icon xoay xoay loading thôi

const HomePage = Loadable({
  loader: () => import('../path/to/components/HomePage'),
  loading: Spinner,
})
const About = Loadable({ loader: () => import('../path/to/components/About'), loading: Spinner })
const KhuyenMai = Loadable({
  loader: () => import('../path/to/components/Promotion'),
  loading: Spinner,
})
// loader có nghĩa là đã load xong sẽ trả về Component trong cái import
// loading có nghĩa đang load soure chưa xong tạm thời load component nào đó tạm. Ở đây là Spinner

const App = () => (
  <Router>
    <div>
      <Link to="/">Trang chủ</Link>
      <Link to="/about">Về chúng tôi</Link>
      <Link to="/promotion">Nhận khuyến mãi ngay</Link>
      <Route exact path="/" component={HomePage} />
      <Route path="/about" component={AboutUs} />
      <Route path="promotion" component={Promotion} />
    </div>
  </Router>
)

ReactDOM.render(<App />, document.getElementById('app'))
```

Xong, bây giờ thì từ 1 file bundle.js nặng 2Mb ta có các file sau chẳng hạn:

    – main.js(File chứa soure thư viện như React, Lodash… và file đầu tiên App.) năng 1Mb
    – homepage.chunk.js nặng 300Kb
    – about.chunk.js nặng 200Kb
    – promotion.chunk.js nặng 500Kb

Khi bạn vào localhost:3000/ thì bạn bật F12 lên(google chorme chế độ cho dev debug)
Bên phần network thì bạn sẽ thấy nó tải main.js 1Mb trước sau đó nó tải tiếp homepage.chunk.js 300Kb sau.
Vậy là ta chỉ cần tải tổng cộng 1.3Mb để vào được trang chủ thay vì 2Mb với cách không sử dụng code splitting(☞ Step 1).

Tiếp, bạn click vào link “Về chúng tôi” thì bạn sẽ thấy network tải thêm phần about.chunk.js 200Kb, Tải xong thì bạn sẽ
thấy nội dung About vơi link localhost:3000/about

Tiếp, bạn quay vê trang chủ và click vào “Nhận khuyến mãi ngay” thì bạn để ý network tải tiếp file promotion.chunk.js 500Kb
Tải xong bạn sẽ thấy nội dung của Promotion với link localhost:3000/promotion

Và bạn có để ý thì các file about.chunk.js, homepage.chunk.js và promotion.chunk.js được file main.js gọi, từ đây bạn cũng
biết thêm một điều rằng file js cũng có thể gọi file js khác chứ không nhất thiết phải sử dụng thẻ < script>

Mình sẽ nói tiếp một tí về phần kinh nghiệm của mình khi spliting ở phần CHIA SẺ THÊM về phần này.

### Production build

Yeah!, nó thật ra đơn giản lắm. File bundle của bạn ban đầu lớn một phần là do code bạn có nhiều comment qúa chẳng hạn,
hoặc tên biến dài, hoặc ký tự Enter thì vô vàn, blabla. WEBPACK thần thánh sẽ giúp bạn minimize code lại.
[Webpack Production Build](https://webpack.js.org/guides/production/)

### gzip

Cái này thì như kiểu như này.
– Browser: Ê Server, bundle.js nặng đấy, gửi tao file nén đi.
– Server: Okay chú, để anh nén đã…. Okay của chú đây.
– Browser: Ukm, lấy được rồi, để tao giải nén rồi chạy lên cho người dùng.

Giả sử file bundle.js của bạn nặng 2Mb đi, khá tốn băng thông. Server sẽ nén lại với khoảng đâu có 300Kb, Browser
sẽ tải về và giải nén ra 2Mb lại thôi. Giải nén xong rồi đọc.

Mình không chắc các trình duyệt cũ có hỗ trợ cái này hay không nữa. Bên server mình dùng express thì mình dùng
kèm compression. Tài liệu:
[Use gzip compression](https://expressjs.com/en/advanced/best-practice-performance.html#use-gzip-compression)

Để check trang web của bạn đã sử dụng gzip chưa thì F12 kiểm tra phần network. Nó có ghi dung lượng đó bạn.

### Server-side-rendering(SSR)

Cái này nó không có gì phức tạp hết, làm thực tế mới khó đấy.
Mình miêu tả nó đơn giản thế này thôi nhé: Mang tiếng là Server-side-rendering nhưng thật chất nó vẫn là
Client-side-rendering đấy, rất là lừa tình. Nó render ra HTML tĩnh trên server rồi gửi cho Browser
nhằm tiết kiệm băng thông thôi, nó vẫn render lại ở client như bình thường thôi à. Vì HTML của một phần nào đó
cho người dùng thấy nên cảm giác nó nhanh chứ thực tế nó lấy đi sức lao động của server nhiều hơn.
[https://reactjs.org/docs/react-dom-server.html](https://reactjs.org/docs/react-dom-server.html)

Ví dụ với server nodejs.

```javascript
import HomePage from '../path/to/components/HomePage' // Ví dụ const HomePage = () => <div>This is HomePage</div>;
app.get('/homepage', (res, req) =>
  res.send(`
        <html>
        <body>
            <div id="app">${ReactDOMServer.renderToString(<HomePage />)}</div>
            <script src="/bundle.js"></script>
        </body>
        </html>
    `)
)
```

Lúc này nếu bạn F12 và check phần network và xem respone của localhost:3000/ bạn sẽ thấy như này

```html
<html>
  <body>
    <div id="app"><div>This is HomePage</div></div>
    <script src="/bundle.js"></script>
  </body>
</html>
```

Khi sử dụng SSR bạn nên sử dụng ReactDOM.hydrate thay cho ReactDOM.render, vì nó sẽ tối đa performace render hơn, vì
dù sau cũng render một phần HTML ở server rồi mà. Việc còn lại chỉ là render ở Browser để có React Application.
Tuy nhiên vì cái này khá khó nên 1, 2 câu không thể nói hết được. Mình sẽ viết ở bài sau nhé.

### Sử dụng CDN

Bài viết này mình đọc sơ qua: [https://techtalk.vn/cdn-chi-1-giay-lam-doi-thay-tam-tri-khach-hang.html](https://techtalk.vn/cdn-chi-1-giay-lam-doi-thay-tam-tri-khach-hang.html)

Nếu bạn ngại đọc thì mình có thể tóm tắt ở ngay đây:
Giả sử server của mình ở VN, những người dùng ở Mỹ sẽ phải request nữa vòng trái đấy để lấy file bundle.js

CDN sẽ giúp những người ở Mỹ request lấy file bundle.js ngay tại Mỹ, ví dụ http://cdn-whatever.com/nvh26041994/bundle.js
chẳng hạn. Vì CDN có cơ sở hạ tầng rộng khắp thế giới nên bạn có thể yên tâm dùng. À mà cũng phaỉ chịu chi tí.

Nếu bạn giàu, cho bundle.js lên CDN.
Nếu bạn giàu mà bạn bạn muốn chơi game không nạp card?. Vẫn chơi CDN được vì những thư viên phổ biến đều có link CDN hết.

### Web worker

Mình sẽ tìm hiểu thêm về thằng này và cập nhật sau nhé.

## MỘT VÀI CHIA SẺ THÊM

– Đối với production build và CDN, bạn nên chia ra các môi trường như development, production cho webpack
webpack.prob.babel.js
webpack.dev.babel.js
– WEBPACK 4 hỗ trợ tách các third-party ra file riêng vendor-main.chunk.js(Cái này chắc cho vào phần code splitting).
webpackConfig: {
…
optimization: {
splitChunks: { chunks: ‘all’ }
}
}
– Về Server-side-rendering mình sẽ có bài riêng về nó sau nhé.
– Khi sử dụng react-loadable, bạn không nên code như mình ở ☞ Step 2 như vậy. Rất khó tái sử dụng.

Với ☞ Step 2 thì mình sẽ refactor như này.

    - components
    - HomePage
    - index.js
    - Loadable.js
    - About
    - index.js
    - Loadable.js
    - Promotion
    - index.js
    - Loadable.js

Khi sử dụng mình chỉ cần

    // recomment cách này
    import HomePage from '../path/to/components/HomePage/Loadable';

Hoặc bạn cũng có thể refactor theo kiểu

    - components
    - HomePage.js
    - About.js
    - Promotion.js
    - Loadable
    - HomePage.js
    - About.js
    - Promotion.js

Khi sử dụng chỉ cần

    import HomePage from '../path/to/components/Loadable/HomePage';

_Nguồn: https://github.com/nguyenvanhoang26041994/dev-experiences/blob/master/React/how_to_make_best_performance_
