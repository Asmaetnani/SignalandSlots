# SignalandSlots
Implements your first Gui Interactive Widget  

## Introduction
In GUI programming, when we change one widget, we often want another widget to be notified. More generally, we want objects of any kind to be able to communicate with one another.To assure this communication we use signals and slots .The idea of signals ans slots is to create a particular "link" between two functions of two independent classes, so that when we call the function of the first object, the function of the second object is automatically called. The first function is called "signal", the second "slot", the link between the two is called a "connection".  
The goal of this of this application is to get familiar with this mechanism and learn how to use it .  
  
## Calculator
This exercise follows up to add interactive functionality to the calculator widgets written in the previous homework. The goal is to use Signals and Slots to simulate a basic calculator behavior. The supported operations are *, +, -, /,√,!,².  
### calculator.h
```cpp
#ifndef CALCULATOR_H
#define CALCULATOR_H

#include <QMainWindow>
#include <QGridLayout>
#include <QVector>
#include <QPushButton>
#include <QLCDNumber>
#include <QString>

class Calculator : public QWidget
{
    Q_OBJECT
public:
    Calculator(QWidget *parent = nullptr);
    ~Calculator();


private:
    double * left=0;          //left operand
 double * right=0;         // right operand
    QString *operation;  // Pointer on the current operation


 

protected:
    void createWidgets();        //Function to create the widgets
    void placeWidget();         // Function to place the widgets
    void makeConnexions();      // Create all the connectivity

//events
protected:
    void keyPressEvent(QKeyEvent *e)override;     //Override the keypress events

private:
    QGridLayout *buttonsLayout; // layout for the buttons
    QVBoxLayout *layout;        //main layout for the button
    QVector<QPushButton*> digits;  //Vector for the digits
    QPushButton *enter;  // enter button
    QPushButton *clear;  //clear button
    QPushButton * fact;  // factorial button
    QPushButton * racinecarre; //square root button
    QPushButton * puissance;   //squared button
    QVector<QPushButton*> operations; //operation buttons
    QLCDNumber *disp;             // Where to display the numbers


public slots:
   void changeOperation();
   void newDigit();
   void enterButton();
   void Clear();
   void factoriel();
   void racine();
   void puissance2();


};
#endif // CALCULATOR_H

```
In this part we declared all the variables,methods and slots we are going to use , such as:  
enterbutton() : to show the final result.  
clear() : to clear the screen.  
newDigit() : to get which button was clicked.  
changeOperation() : to handle the click on operations.  
  
### Calculator.cpp
```cpp
#include "calculator.h"
#include <QKeyEvent>
#include <QApplication>
#include<math.h>


Calculator::Calculator(QWidget *parent)
    : QWidget(parent)
{
    createWidgets();
    placeWidget();
    makeConnexions();
    left=nullptr;
    right=nullptr;
    operation=nullptr;


}

Calculator::~Calculator()
{
    delete disp;
    delete layout;
    delete buttonsLayout;
    delete left;
    delete right;
    delete operation;

}


void Calculator::createWidgets()
{
    //Creating the layouts
    layout = new QVBoxLayout();
    layout->setSpacing(2);

    //grid layout
    buttonsLayout = new QGridLayout;


    //creating the buttons
    for(int i=0; i < 10; i++)
    {
        digits.push_back(new QPushButton(QString::number(i)));
        digits.back()->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Fixed);
        digits.back()->resize(sizeHint().width(), sizeHint().height());
    }
    //enter button
    enter = new QPushButton("Enter",this);
    enter->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Fixed);
    enter->resize(sizeHint().width(), sizeHint().height());

    clear = new QPushButton("Clear",this);
    clear->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Fixed);
    clear->resize(sizeHint().width(), sizeHint().height());


    fact = new QPushButton("Fact",this);
    fact->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Fixed);
    fact->resize(sizeHint().width(), sizeHint().height());

    racinecarre = new QPushButton("√",this);
    racinecarre->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Fixed);
    racinecarre->resize(sizeHint().width(), sizeHint().height());

    puissance = new QPushButton("x²",this);
    puissance->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Fixed);
    puissance->resize(sizeHint().width(), sizeHint().height());




    //operatiosn buttons
    operations.push_back(new QPushButton("+"));
    operations.push_back(new QPushButton("-"));
    operations.push_back(new QPushButton("*"));
    operations.push_back(new QPushButton("/"));


    //creating the lcd
    disp = new QLCDNumber(this);
    disp->setDigitCount(6);

}

void Calculator::placeWidget()
{

    layout->addWidget(disp);
    layout->addLayout(buttonsLayout);


    //adding the buttons
    for(int i=1; i <10; i++)
        buttonsLayout->addWidget(digits[i], (i-1)/3, (i-1)%3);


    //Adding the operations
    for(int i=0; i < 4; i++)
        buttonsLayout->addWidget(operations[ i], i, 4);


    //Adding the 0 button
    buttonsLayout->addWidget(digits[0], 3, 0);
    buttonsLayout->addWidget(enter, 3, 1, 1,2);
    buttonsLayout->addWidget(clear);
    buttonsLayout->addWidget(fact);
    buttonsLayout->addWidget(racinecarre);
    buttonsLayout->addWidget(puissance,4,4);

    setLayout(layout);
}

void Calculator::makeConnexions()
{for(int i=0; i <10; i++)
        connect(digits[i], &QPushButton::clicked,this, &Calculator::newDigit);
  for(int j=0; j <4; j++)
          connect(operations[j], &QPushButton::clicked,
                  this, &Calculator::changeOperation);

   connect(enter,&QPushButton::clicked,this,&Calculator::enterButton);
   connect(clear,&QPushButton::clicked,this,&Calculator::Clear);
   connect(fact,&QPushButton::clicked,this,&Calculator::factoriel);
   connect(racinecarre,&QPushButton::clicked,this,&Calculator::racine);
   connect(puissance,&QPushButton::clicked,this,&Calculator::puissance2);
}

void Calculator::newDigit(){
    //getting the sender
        auto button = dynamic_cast<QPushButton*>(sender());

        //getting the value
        double value = button->text().toDouble();

        //Check if we have an operation defined
        if(operation)
        {
            //check if we have a value or not
            if(!right)
                right = new double{value};
            else
                *right = 10 * (*right) + value;

            disp->display(*right);

        }
        else
        {
            if(!left)
                left = new double{value};
            else
                *left = 10 * (*left) + value;

            disp->display(*left);
        }


}

void Calculator::changeOperation()
{
    //Getting the sender button
    auto button = dynamic_cast<QPushButton*>(sender());

    //Storing the operation
    operation = new QString{button->text()};

    //Initiating the right button
    right = new double{0};

    //reseting the display
    disp->display(0);
}

void Calculator::enterButton(){

        if(this->operation==QString("+"))
        disp->display((*left)+(*right));

        if(this->operation==QString("*"))
        disp->display((*left)*(*right));

        if(this->operation==QString("-"))
        disp->display((*left)-(*right));

        if(this->operation==QString("/"))
        disp->display((*left)/(*right));



}
void Calculator::Clear()
{
        right=0;
        left=0;
        operation=nullptr;
        disp->display(0);
}
void Calculator::factoriel(){
    for(int i=*left-1; i>0; i--)
                (*left) *= i;
            disp->display(*left);

}
void Calculator::racine(){
    left = new double{sqrt(*left)};
            disp->display(*left);
}
void Calculator::puissance2(){
    left = new double{ (*left) * (*left)};
            disp->display(QString::number(*left));
}

void Calculator::keyPressEvent(QKeyEvent *e)
{
    //Exiting the application by a click on space
    if( e->key() == Qt::Key_Escape)
        qApp->exit(0);


}

```
### main.cpp
```cpp
#include "calculator.h"

#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    Calculator w;
    w.setWindowTitle("Calculator");
    w.resize(500,500);
    w.show();
    return a.exec();
}

```
### Output


## Traffic Light
In this exercise, we will use the QTimer to simulate a traffic light.  
![image](https://user-images.githubusercontent.com/86807044/142773203-2179610a-bb6a-41fc-a690-d7306dbd256c.png)   

First of all here is the header:
### trafficlight.h
```cpp
#ifndef TRAFFIC_LIGHT_H
#define TRAFFIC_LIGHT_H

#include <QWidget>
#include<QVector>
#include<QKeyEvent>

class QRadioButton;

class TrafficLight: public QWidget{
  Q_OBJECT

public:

  TrafficLight(QWidget * parent = nullptr);

protected:
     void createWidgets();
     void placeWidgets();
     void timerEvent(QTimerEvent *e)override;
     void keyPressEvent(QKeyEvent*e)override;

private:

  QRadioButton * redlight;
  QRadioButton * yellowlight;
  QRadioButton * greenlight;
  QVector<QRadioButton*>lights;
  int index;
  int lifetime;
};


#endif
```


We are going to try three different versions of this exercise:  
#### First one:  
  .If the red light is activated->activate the yellow one .  
  .If the yellow light is activated->activate the green one .  
  .If the green light is activated-> activate the red one .  
  
  
Here is the implementation for it:  

```cpp
//si le feu rouge est active -> activer le jaune
    if(redlight->isChecked())
        yellowlight->toggle()
//si le feu jaune active-> activer le vert

      else if(yellowlight->isChecked())
        greenlight->toggle();
//si le feu vert active->activer le rouge

      else if(greenlight->isChecked())
       redlight->toggle();

```

 #### Second one:
   .The red light should be activated for 4 seconds.  
   .The yellow light should be acttivated for 1 second.  
   .The green light should be activated for 2 seconds.  
   
   Here is the implementation for it:
   ```cpp
   lifetime++;

    if(redlight->isChecked() && lifetime==4){
        yellowlight->toggle();
    lifetime=0;}

    else if(yellowlight->isChecked() && lifetime==1){
        greenlight->toggle();
    lifetime=0;}

    else if(greenlight->isChecked() && lifetime==2){
        redlight->toggle();
    lifetime=0;}
   ```
   #### Third one
   .If R button is pressed the red light should be activated.  
   .If Y button is pressed the yellow light should be activated.  
   .If G button is pressed the green light should be activated.  
   
    Here is the implementation for it:  
```cpp
     void TrafficLight::keyPressEvent(QKeyEvent *e){
  if(e->key()==Qt::Key_Escape)
      qApp->exit();

    else if(e->key()==Qt::Key_R)
       redlight->toggle();

    else if(e->key()==Qt::Key_Y)
        yellowlight->toggle();

    else if(e->key()==Qt::Key_G)
        greenlight->toggle();

}
```
### Output


## Conclusion
This session was very interesting especially because we had to deal with a lot of segmentation faults, so we learned a lot of new things and we had fun as well . 


    
    
  >Team : Aya Benlahbib and Asmae Tnani
   
   








