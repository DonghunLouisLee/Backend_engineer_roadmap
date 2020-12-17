# Metadata
author: Louis Lee   
date: 20201215  
topic: rust - rc and refcell  

# Introduction

러스트의 소유권 규칙

# Summary

Rc: reference counter
일종의 스마트 포인터로써 값에 대한 참조의 개수를 추적해서 더 이상 값에 대한 참조한것들이 없을때 데이터를 메모리에서 해제할수 있게 해주는 장치이다. 당연하게 Rc<T> 는 heap에 저장되며 소유권이 적용되는 범위는 컴파일 시간에 알 수 있다. 아주 기본적인 예시로는

```

enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    // a 는 값이 아니라 Cons(5, ...)에 대한 포인터이다.
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}

```
근데 만약 Rc<T>로 T를 감싸버리면 T를 바꾸고 싶을때는 어떻게 하면 좋을까? 참조되고 있는 값이 바뀌어버리면 문제가 생기지 않을까? 이 문제를 해결하기 위해 우리는 RefCell을 사용한다! 

RefCell<T> 는 Rc<T>와는 달리 단 하나만의 소유권을 인정한다. Box<T>와 비슷한 느낌이지만 다른 점은 Box<T>는 소유권은 컴파일 타임에 결정하고 RefCell<T>은 runtime에 결정한다는 차이가 있다. 즉, RefCell<T>를 쓴다면 러스트의 훌륭하고 아름다운 컴파일러의 득을 보지는 못한다. 성능 측면에서도 컴파일 타임에 결정이 되는 Box<T>가 유리한데 그럼에도 불구하고 RefCell<T>가 사용되는 경우는 아래와 같다.

1. Mock object

바로 예시를 통해 보자.

```
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: 'a + Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
    where T: Messenger {
        pub fn new(messenger: &T, max: usize) -> LimitTracker<T> {
            LimitTracker {
                messenger,
                value: 0,
                max,
            }
        }

        pub fn set_value(&mut self, value: usize) {
            self.value = value;
          
            let percentage_of_max = self.value as f64 / self.max as f64;

            if percentage_of_max >= 1.0 {
                self.messenger.send("Error: You are over your quota!");
            } else if percentage_of_max >= 0.5 {
                self.messenger.send("Urgent warning: You've used up over 90% of your quota!");
            }
        }
    }
```
이 LimitTracker는 메센저의 메세지가 최대 용량의 몇 퍼센트인지에 따라서 알림 문자를 보내주는 기능을 한다. Messenger trait 을 구현하고 있는 어떠한 struct 도 모두 이를 limitracker를 사용할수 있다. 여기서 
set_value(&mut self)를 주목하라. set value 함수를 호출할때 우리는 mutable reference를 빌려온다. 즉, 이 함수가 불리는 동안에는 우리는 다른 가변 참조를 할수 없다. 이 사실을 기억하고 우리가 여기에 대해 테스트 코드를 짜려고 하는데 refcell을 사용하지 않고서는 이러한 형식으로 쓸수 있다. 

```
#[cfg(test)]
mod tests {
    use super::*;

    struct MockMessenger {
        sent_message: Vec<String>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_message: vec![],
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_message.push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_message.len(), 1);
    }
}
```
무언가가 이상하지 않나? 그렇다 자 여기서 mockmessenger는  sent_message: Vec<String> 을 field로 가진다. 여기서 send 함수를 잘 보면 &self 를 사용하는데 당연하게 &mut self 가 아닌 그냥 &self로는 내부 field인 sent_messages를 변경할수 없다. 그렇다고 &mut self를 쓰게 된다면? 위에서 우리가 보았듯이 set_value함수가 &mut self를 가지기 때문에 가변참조를 두번하게 되서 컴파일 에러가 발생하다. 바로 이때 우리는 refcell을 쓰면 이 문제를 피해갈수 있다. 

```
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_message: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_message: RefCell::new(vec![]),
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_message.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_message.borrow().len(), 1);
    }
}
```
여기서처럼 sent_message를 Vec<String>이 아닌 RefCell로 <Vec<String>>을 감싸버리면 우리는 간단히 가변참조가 필요한 부분에서만 borrow_mut()를 사용해서 컴파일 에러를 피해갈수 있다. 물론 실제로 production level에서는 이 방법 대신 다른 방법을 사용하겠지만 적어도 코드를 테스트 해보기 위해서 우리는 refcell을 통해 컴파일러를 속일수 있다. 

번외편 
Rc<RefCell<T>> vs RefCell<Rc<T>>? 

Rc<RefCell<T>>:  the whole thing is shared and each shared owner gets to mutate the contents. The effect of mutating the contents will be seen by all of the shared owners of the outer Rc because the inner data is shared.

RefCell<Rc<T>>: In order to mutate the inner data, you would need to mutably borrow from the outer RefCell, but then you'd have access to an immutable Rc. The only way to mutate it would be to replace it with a completely different Rc.

결론: RefCell<Rc<T>>는 진짜...진짜 쓸데가 없다!
# Notes
