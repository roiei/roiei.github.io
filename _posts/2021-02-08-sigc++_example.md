---
title: "sigc example"
date: 2020-02-08 00:00:00 -0400
categories: C++
---

## define signal and connect it to a function

&nbsp;

```c++
class XXDetector {
public:
    ...
    sigc::signal<void> XX_signal;
};


void XXDetector::checkXX() {
    XX_signal.emit();
}


void XX_slot_func() {
    cout<<"XX signal is detected"<<endl;
}


int testXXDetector() {
    XXDetector detector;
    detector.XX_signal.connect(sigc::ptr_fun(XX_slot_func));
    ...
    detector.checkXX();
}
```

&nbsp;

```c++
class XXDetector {
public:
    ...
    sigc::signal<void> XX_signal;
    std::string m_status;
};


void XXDetector::checkXX() {
    XX_signal.emit(m_status);
}


class XXObserver {
public:
    ...
    void XX_slot_func(const std::string& msg) {
        cout<<msg<<endl;
    }
}


int testXXDetector() {
    XXDetector detector;
    XXObserver observer1;
    XXObserver observer2;

    sigc::connection conn1 = detector.XX_signal.connect(sigc::mem_fun(observer1, &observer::XX_slot_func));
    detector.XX_signal.connect(sigc::mem_fun(observer2, &observer::XX_slot_func));

    ...
    detector.checkXX();  // expect 2 observers receive message

    conn1.disconnect();

    detector.checkXX(); // expect 1 observer receives message
}
```