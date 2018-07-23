---
title: JavaFx苦逼之路
date: '2018-1-12 09:42'
categories: JavaFx
tags:
  - JavaFx
abbrlink: 69cc9e87
---

**啊啊啊,JavaFX贼难玩,不喜欢**
<!-- more -->

# 前言

目前我能用javafx做的东西都很少,还跟在大佬屁股后面学习.说不定写着写着就脱坑停更!

## 绑定tableView
- 需求是绑定几个测试参数到`tableView`下如图

![1.jpg-11.6kB][1]


- 结构为:`tabPane` -> `Tab` -> `tableView` -> 若干个`TableColumn`

### **第一步先设置`fx:id`**

`tableView`上肯定需要`fx:id`,然后在给每一列绑定`fx:id`,如下图

![2.jpg-14kB][2]![3.jpg-51.7kB][3]

### **第二步,编写相关联的实体**

由于我的`tableview`中显示的数据封装在了实体类中,所以这里我们需要先定义实体,实体的定义如下:

```java
import javafx.beans.property.DoubleProperty;
import javafx.beans.property.IntegerProperty;
import javafx.beans.property.SimpleDoubleProperty;
import javafx.beans.property.StringProperty;

/**
 * 限价委托实体类
 */
public class LimitEntrustEntity extends Commission{

    private DoubleProperty orderState = new SimpleDoubleProperty(); //已成交数量

    private DoubleProperty priceOfOrder = new SimpleDoubleProperty(); //成交均价

    private DoubleProperty gvm = new SimpleDoubleProperty(); //成交总额

    public LimitEntrustEntity(DoubleProperty orderState, DoubleProperty priceOfOrder, DoubleProperty gvm) {
        this.orderState = orderState;
        this.priceOfOrder = priceOfOrder;
        this.gvm = gvm;
    }

    public LimitEntrustEntity() {

    }

    public double getOrderState() {
        return orderState.get();
    }

    public DoubleProperty orderStateProperty() {
        return orderState;
    }

    public void setOrderState(double orderState) {
        this.orderState.set(orderState);
    }

    public double getPriceOfOrder() {
        return priceOfOrder.get();
    }

    public DoubleProperty priceOfOrderProperty() {
        return priceOfOrder;
    }

    public void setPriceOfOrder(double priceOfOrder) {
        this.priceOfOrder.set(priceOfOrder);
    }

    public double getGvm() {
        return gvm.get();
    }

    public DoubleProperty gvmProperty() {
        return gvm;
    }

    public void setGvm(double gvm) {
        this.gvm.set(gvm);
    }

    @Override
    public double getCommissionQty() {
        return super.getCommissionQty();
    }

    @Override
    public DoubleProperty commissionQtyProperty() {
        return super.commissionQtyProperty();
    }

    @Override
    public void setCommissionQty(double commissionQty) {
        super.setCommissionQty(commissionQty);
    }

    @Override
    public double getCommissionTime() {
        return super.getCommissionTime();
    }

    @Override
    public DoubleProperty commissionTimeProperty() {
        return super.commissionTimeProperty();
    }

    @Override
    public void setCommissionTime(double commissionTime) {
        super.setCommissionTime(commissionTime);
    }

    @Override
    public double getCommissionPrice() {
        return super.getCommissionPrice();
    }

    @Override
    public DoubleProperty commissionPriceProperty() {
        return super.commissionPriceProperty();
    }

    @Override
    public void setCommissionPrice(double commissionPrice) {
        super.setCommissionPrice(commissionPrice);
    }

    @Override
    public int getState() {
        return super.getState();
    }

    @Override
    public IntegerProperty stateProperty() {
        return super.stateProperty();
    }

    @Override
    public void setState(int state) {
        super.setState(state);
    }

    @Override
    public String getOrderSource() {
        return super.getOrderSource();
    }

    @Override
    public StringProperty orderSourceProperty() {
        return super.orderSourceProperty();
    }

    @Override
    public void setOrderSource(String orderSource) {
        super.setOrderSource(orderSource);
    }
}
```

**这里我继承了父类,父类如下:**

```java
package com.yczr.threebody.pc.entity;


import javafx.beans.property.*;

import java.io.Serializable;

public class Commission implements Serializable {
    private DoubleProperty commissionQty = new SimpleDoubleProperty();    //委托数量 2
    private DoubleProperty commissionTime = new SimpleDoubleProperty();   //委托时间 2
//    private DoubleProperty triggerPrice;     //触发价格 1
    private DoubleProperty commissionPrice = new SimpleDoubleProperty();  //委托价格 2
//    private TradeStatus realTriggerPrice; //真实触发价 1
    private IntegerProperty state = new SimpleIntegerProperty(); //状态 2
    private StringProperty orderSource = new SimpleStringProperty(); //订单来源 2


    public double getCommissionQty() {
        return commissionQty.get();
    }

    public DoubleProperty commissionQtyProperty() {
        return commissionQty;
    }

    public void setCommissionQty(double commissionQty) {
        this.commissionQty.set(commissionQty);
    }

    public double getCommissionTime() {
        return commissionTime.get();
    }

    public DoubleProperty commissionTimeProperty() {
        return commissionTime;
    }

    public void setCommissionTime(double commissionTime) {
        this.commissionTime.set(commissionTime);
    }

//    public double getTriggerPrice() {
//        return triggerPrice.get();
//    }
//
//    public DoubleProperty triggerPriceProperty() {
//        return triggerPrice;
//    }
//
//    public void setTriggerPrice(double triggerPrice) {
//        this.triggerPrice.set(triggerPrice);
//    }

    public double getCommissionPrice() {
        return commissionPrice.get();
    }

    public DoubleProperty commissionPriceProperty() {
        return commissionPrice;
    }

    public void setCommissionPrice(double commissionPrice) {
        this.commissionPrice.set(commissionPrice);
    }

    public int getState() {
        return state.get();
    }

    public IntegerProperty stateProperty() {
        return state;
    }

    public void setState(int state) {
        this.state.set(state);
    }

    public String getOrderSource() {
        return orderSource.get();
    }

    public StringProperty orderSourceProperty() {
        return orderSource;
    }

    public void setOrderSource(String orderSource) {
        this.orderSource.set(orderSource);
    }
}
```

顺便说一下这里遇到的**坑!!!!!**

1. 定义需要绑定的值时,属性还是用的以前的类型,比如Double类型还是用的Double,但是在javafx中Double变为了javafx中的封装类型DoubleProperty,其它的数据类型应该也是这样;
2. 实体类我用的随机数进行赋值,代码如下:

```java
public static Double getRandom(int max){
        double v = new Random(max).nextDouble();
        return v;
    }

    public static Double getRandom(){
        Double random = getRandom(1000);
        return random;
    }

    public static LimitEntrustEntity getLeEntity() {
        LimitEntrustEntity leEntity =  new LimitEntrustEntity();
        leEntity.setCommissionPrice(getRandom());
        leEntity.setCommissionQty(getRandom());
        leEntity.setCommissionTime(getRandom());
        leEntity.setGvm(getRandom());
        leEntity.setOrderSource(getRandom().toString());
        leEntity.setOrderState(getRandom());
        leEntity.setPriceOfOrder(getRandom());
        leEntity.setState(getRandom().intValue());
        return leEntity;

    }
```

其中的**坑**是真的多!,我没有没父类`Commission`中的属性设置默认值,导致加载数据时为空

![4.jpg-33.7kB][4]



---
### **第三步,在代码中与`fx:id`绑定**

虽然在`fxml文件`中设置了`fx:id`,但是在实际的程序中并没有做任何相关联的绑定,所以这一步就需要进行相关的绑定;

- 定义数据源

```java
private ObservableList<TradeOverviewEntity> exchangeData = FXCollections.observableArrayList();
```

- 绑定`TableView`

```java
@FXML
private TableView<TradeOverviewEntity> exTable;
```

- 绑定列`TableColumn`

```java
@FXML
private TableColumn<TradeOverviewEntity, String> coinName;
@FXML
private TableColumn<TradeOverviewEntity, Number> exPrice;
@FXML
private TableColumn<TradeOverviewEntity, Number> exMaxPrice;
    
//..........省略其它列
```

### 加载数据

- 前面虽然绑定了数据源,但是并没有进行实际的赋值,下面先把值和列对应上

```java
 private void initBean(){
    lpwtTime.setCellValueFactory(cellData -> cellData.getValue().commissionTimeProperty()); //委托时间
    plNumber.setCellValueFactory(cellData -> cellData.getValue().commissionQtyProperty()); //委托数量
    plDeal.setCellValueFactory(cellData -> cellData.getValue().orderStateProperty()); //已成交
    plPrice.setCellValueFactory(cellData -> cellData.getValue().commissionPriceProperty()); //委托价格
    plPrice2.setCellValueFactory(cellData -> cellData.getValue().priceOfOrderProperty()); //成交均价(QC)
    plTotal.setCellValueFactory(cellData -> cellData.getValue().commissionTimeProperty()); //成交总额(QC)
    plState.setCellValueFactory(cellData -> cellData.getValue().stateProperty()); //状态
    plSource.setCellValueFactory(cellData -> cellData.getValue().orderSourceProperty()); //状态
}
```

- 对数据进行赋值

```java
 private void loadData(){
        //随机生成5个订单
        for (int i = 0; i < 5; i++) {
            limitEntrusData.add(BeanUtils.getLeEntity());
        }
    }
```

- 初始化加载数据

```java
@FXML
private void initialize() {
    initBean();
    loadData();
    lpTable.setItems(limitEntrusData);

}
```

### 效果

![5.jpg-39.5kB][5]


## 监听TextField变化

目的是为了完成界面简单的密码强度判断,如下界面

![1.jpg-14.7kB][6]

### 绑定fxid

![2.jpg-24.5kB][7]

其它的fxid就不一一绑定了

### 编写监听方法

```java
public void testyy() {
    password.textProperty().addListener(new ChangeListener<String>() {
        @Override
        public void changed(ObservableValue<? extends String> observable, String oldValue, String newValue) {
        //下面是逻辑处理部分
            if (password.getText().equals("") || password.getText().length() < 1) {
                pwIntensity.setText("登录密码不能为空");
            } else {
                String intensity = null;
                if (password.getText().length() > 5) {
                    intensity = "强";
                } else if (password.getText().length() < 5) {
                    intensity = "弱";
                } else {
                    intensity = "极强";
                }
                pwIntensity.setText("密码强度" + intensity);
            }
        }
    });
}
```

### 调用监听

监听需要在页面初始化的时候就进行加载,所以在`initialize()`中吊起

```java
@FXML
private void initialize() {
    testyy();
}
```

### 效果

![3.jpg-6.2kB][8]


  [1]: http://static.zybuluo.com/pockadmin/bw25kqrop54612olqrssvntv/1.jpg
  [2]: http://static.zybuluo.com/pockadmin/91qwlg7tvmpfwd0z30hv747q/2.jpg
  [3]: http://static.zybuluo.com/pockadmin/gf6ab5fnxb3xl39z2urqb7es/3.jpg
  [4]: http://static.zybuluo.com/pockadmin/hzmd0umdqn2nujbxq3pev4rf/4.jpg
  [5]: http://static.zybuluo.com/pockadmin/dwsczz1ld6y4y3k7et18xxyp/5.jpg
  [6]: http://static.zybuluo.com/pockadmin/vm29nmgdmozl83z0ibtwwlri/1.jpg
  [7]: http://static.zybuluo.com/pockadmin/w7y1z334fpiu64nn38is3prb/2.jpg
  [8]: http://static.zybuluo.com/pockadmin/x7mqcccs1awmgfu5gedm90xb/3.jpg