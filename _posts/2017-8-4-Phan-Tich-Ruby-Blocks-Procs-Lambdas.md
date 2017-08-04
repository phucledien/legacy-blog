---
layout: post
title: "Phân tích Ruby: BLocks, Procs, Lambdas hay còn gọi là \"Closures\""
---
*Bài viết sau sẽ giải thích cặn kẽ các khái niệm quen thuộc về closures trong Ruby như BLocks và Procs, cũng như các khái niệm ít được nhắc đến như Lambdas và Methods.*

## Blocks

Có lẽ một trong những phần khó hiểu nhất trong việc học Ruby chính là hiểu được cặn kẽ (khoảnh khắc AHA!) blocks là gì và chúng hoạt động thế nào? Đối với các bạn đã từng sử dụng những ngôn ngữ lập trình khác thì khái niệm này có vẻ mới nên các bạn chưa quen, nhưng thật ra nó thật sự khá đơn giản. Và chắc chắn là bạn đã gặp blocks rất nhiều lần khi sử dụng các hàm `#each` hay `#map` trong Ruby.
<!-- more -->
Blocks là một đoạn code mà bạn có thể bỏ vào một hàm nào đó dưới dạng "input". Nó thường được gọi là "anonymous functions" (hàm bí danh) bởi vì chúng không có tên nhưng nó hoạt động như một hàm bình thường vậy. Bạn có thể nghĩ nó như một hàm helper nào đó, blocks chỉ xuất hiện khi bạn gọi một hàm nào đó như là `#each` chẳng hạn.

Bạn khai báo một block bằng cách sử dụng cặp ngoặc nhọn `{}` khi nó chỉ có một dòng, hoặc là `do ... end` nếu nó có nhiều dòng (bạn thích sử dụng cách nào cũng được):

```ruby
> [1,2,3].each { |num| print "#{num}! " }
1! 2! 3! =>[1,2,3]
> [1,2,3].each do |num|
>    print "#{num}!"
> end
1! 2! 3! =>[1,2,3]         # Hai cách gọi đều có kết quả như nhau cả
``` 

Cũng như hàm, blocks cũng có thể nhận tham số đầu vào, trả về giá trị. BLocks cho phép bạn sử dụng *return* một cách ngầm định (nó sẽ tự trả về giá trị của cái gì năm ở dòng cuối cùng) nhưng bạn KHÔNG NÊN `return`, vì nó sẽ return luôn cái hàm đang gọi cái block đó. 

Blocks được sử dụng như một tham số đầu vào cho một hàm (vd như là hàm `#each`), nó cũng y chang mấy cái tham số mà bạn hay bỏ vào trong dấu `()` khi gọi hàm thôi nhưng nó luôn được truyền vào phía cuối khi gọi hàm vì nó tốn khá nhiều dòng. Bạn không cần nghĩ quá nhiều về nó, cũng như vậy, hàm `#each` cũng chẳng có gì đặc biệt. Nó được viết để có thể nhận block như một tham số đầu vào.

Vậy làm thế nào mà hàm `#each` có thể nhận vào một block? Đó là nhờ vào ma thuật của `yield`, yield cơ bản sẽ có nhiệm vụ báo cho block rằng "mày vào đây thế chỗ hộ tao cái". Khi bạn muốn viết một hàm nhận vào một block bạn cũng chẳng cần khai báo nó ở trong dấu `()` như viết các hàm bình thường khác. Chỉ cần trong hàm của bạn có gọi `yield` thì mặc định Ruby sẽ tự hiểu bạn muốn truyền vào hàm một block. 

`yield` cũng có thể chuyền các tham số vào block. Để dễ hiểu hơn thì mình sẽ minh họa bằng cách viết lại một hàm `#my_each` có chức năng tương tự `#each` cho class Array. Qua đó, chúng ta có thể sử dụng hàm `#my_each` này cho một mảng nào đó qua cú pháp như sau `[1, 2, 3].my_each` thay vì bỏ nó một mảng vào làm tham số `my_each([1, 2, 3])`:

```ruby
class Array 
  def my_each
    i = 0
    while i < self.size
        yield(self[i])  
        i+=1      
    end
    self
  end
end
```

Như các bạn thấy, chúng ta sẽ duyệt qua từng phần từ trong mảng đã gọi hàm `#my_each` (bằng cách dùng self). Sau đó chúng ta gọi một block nào đó được truyền vào hàm `#my_each`. Chúng ta truyền vào block đó các phần tử trong mảng mà chúng ta duyệt qua. Cuối cùng return lại mảng gốc. Chúng ta có thể gọi hàm này tương tự như `#each`:

```ruby
> [1,2,3].my_each { |num| print "#{num}!" }
1! 2! 3! => [1,2,3]
```

Những gì diễn ra tương tự như sau:

```ruby
class Array
  def my_each
    i = 0
    while i < self.size
        print "#{self[i]}!"   # block đã thế chỗ cho yield
        i+=1
    end
    self
  end
end
```

Nếu như chúng ta muốn viết một hàm dạng như `#each` thì block là thứ chúng ta không thể bỏ qua. Một trường hợp khác để sử dụng block đó là khi chúng ta muốn viết các hàm có thể "override" lại cách mà hàm đó hoạt động bằng block truyền vào, hàm `#sort` là một ví dụ. Chúng ta hoàn toàn có thể sắp xếp các phần tử theo cách mà chúng ta muốn bằng cách truyền vào block.

Nếu bạn muốn kiểm tra block có được truyền vào hàm hay không, bạn có thể sử dụng hàm `#block_given?`, ví dụ: `yield if block_given?`

Ok! Vậy nếu bạn muốn truyền 2 blocks vào hàm thì phải làm thế nào? Hay là bạn muốn lưu block của bạn lại để dành dùng lần sau? Đó là công việc cho *Procs*, hay còn được gọi là Procedures!

## Procs

Thật ra block chính là một Proc (nói cách khác Proc là tên class của một block) và họ đặt tên vậy chỉ để làm khó bạn mà thôi :)). Bạn có thể tưởng tượng kiểu như block là một phiên bản tạm và trần trụi hơn so với Proc, và Ruby thêm vào chỉ để bạn dễ dàng hơn khi làm việc với các hàm như `#each`.

Một Proc chính là một block mà được lưu lại vào một biến nào đó.

`> my_proc = Proc.new { |arg1| print "#{arg1}! " }`

Bạn có thể sử dụng biến `my_proc` này để chuyền vào một hàm nào đó, bằng cách thêm dấu `&` phía trước:

```ruby
> [1,2,3].each(&my_proc)
1! 2! 3! =>[1,2,3]
```

Nó cũng khá giống với việc bạn truyền vào một block.

Khi bạn tạo một hàm nhận vào nhiều proc thì bạn cần phải thay đổi một chút bằng cách sử dụng `#call` thay vì `yield` (bởi vì nếu sử dụng yield thì làm sao Ruby biết bạn muốn chạy Proc nào?). `#call` có nhiệm vụ chạy Proc đã gọi nó. Bằng có thể chuyền tham số cho Proc bằng cách chuyền vào hàm hàm `#call`:

` my_proc.call("howdy ")  # howdy => nil`

Trong hầu hết trường hợp, các bạn chỉ cần sử dụng block là đủ, đặc biệt là trong giai đoạn đầu của project. Một khi bạn thấy chỗ nào cần thiết để sử dụng Proc (như là chuyền vào nhiều block vào hàm hay bạn muốn lưu block lại để dùng sa này), thì Procs sẽ là lựa chọn tuyệt vời cho bạn.

Blocks và Procs đều là một dạng của "closure". Closure là cách nói theo kiểu sang chảnh, đầy tính khoa-học-máy-tính cho một đoạn code mà bạn có thể đưa vào mọi nơi, nhưng bạn phải gán nó vào biến khi gọi nó lần đầu tiên. Nó là một khái niệm bao gồm cả blocks lẫn Procs và...

Trong Ruby có thêm hai khái niệm tương tự như closures cũng rất đáng quan tâm, nhưng bạn không cần trở thành chuyên gia về nó vì nó chỉ được sử dụng trong một vài dạng ứng dụng. Đầu tiên là *lambda*.

## Lambdas

Nếu ta nói Procs là một phiên bản rộng hơn của blocks, thì cũng tương tự, lambdas chính là một phiển bản rộng hơn của Procs. Nó càng ngày bước gần hơn đến khía niệm methods (hàm), nhưng vẫn chỉ được coi là một anonymous function. Nếu bạn từng sử dụng Javascript thì chắc chắn quá quen thuộc với khái niệm này.

Bạn chỉ cần tập trung vào sự khác biệt giữa lambdas và Procs, lambda hoạt động gần như một hàm thực thụ. Điều này có nghĩa là gì?

+ Một lambda sẽ giúp bạn linh hoạt hơn trong việc return giá trị (ví dụ như bạn muốn return nhiều giá trị cùng lúc) vì bạn hoàn toàn an toàn khi sử dụng `return` trực tiếp trong lambda. Với lambdas, `return` sẽ chỉ giá về giá trị và thoát ra khỏi lambda và tiếp tục thực hiện các dòng code trong hàm mà nó được chuyền vào, chứ không phải trả về giá trị và thoát ra luôn hỏi hàm cha (điều này sẽ xảy ra với Procs như mình đã đề cập bên trên).
+ Lambdas sẽ nghiêm khắc hơn trong việc kiểm tra số lượng tham số mà bạn chuyền vào nó (chương trình sẽ báo lỗi nếu như bạn chuyền vào sai lượng tham số).

Đây là một ví dụ đơn giản về syntax của lambda

```ruby
 lambda do |word| 
>   puts word
>   return word            # Bạn không thể làm điều này với Procs
> end.call("howdy ")
howdy => "howdy " 
```

Và closure cuối cùng  chính là *Method*

## Method

Method chính là closure gần nhất với chính khái niệm method =] (so với blocks, Procs, lambdas). Method được với hoa vì đây là tên một class. Nó cũng chỉ là một cách viết khác của một hàm bình thường mà thôi. Chúng ta sẽ sử dụng symbol để chuyền Method vào trong một hàm khác bằng hàm `#method`. Có lẽ ví dụ thì sẽ dễ hiểu hơn.

```ruby
class Array
  def my_each(some_method)
    i = 0
    while i < self.size
      some_method.call(self[i])
      i+=1
    end
  end
  self
end

def print_stuff(word)
  print "#{word}! "
end

> [1,2,3].my_each(method(:print_stuff))    # symbolize the name!
1! 2! 3! => nil
```

Tóm lại:

+ *Blocks* là một đoạn code không có tên mà bạn có thể chuyền vào một hàm khác. Có thể sử dụng bất cứ lúc nào.
+ *Procs* tương tự như blocks nhưng bạn có thể lưu chúng trong các biến, điều này sẽ giúp bạn chuyền vào các hàm một cách rõ ràng cũng như có thể lưu lại và lấy ra dùng khi cần. Được dùng cũng tương đối.
+ *Lambdas* là một hàm đầy đủ nhưng không có tên. Hiếm dùng.
+ *Methods* được dùng để chuyền vào một hàm khác qua cách gọi hàm method và symbolize tên hàm. Hiếm dùng.
+ *Closure* là cái ô che hết tất cả các khái niệm ở trên, nó liên quan đến việc chuyền một vài đoạn code vào hàm bằng cách nào đó.
