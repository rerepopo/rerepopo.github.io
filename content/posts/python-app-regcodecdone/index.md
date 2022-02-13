+++
title = "Python App regCodecDone 20200607"
date = 2020-06-07T19:47:31+08:00
# description = "" # Used by description meta tag, summary will be used instead if not set or empty.
featured = false
comment = false
toc = true
reward = true
pinned = false
categories = [
  "python"
]
tags = [
  ""
]
series = [
  ""
]
images = []
+++




## PycharmProjects\regDecoder 20200607 
pip install pyinstaller
pyinstaller -Fw regCodec.py
![](https://i.imgur.com/ytT279x.png)

https://github.com/rerepopo/regCodecDone
<!--more-->
### regCodec.py
```
# -*- coding: utf-8 -*-
from PyQt5 import QtWidgets, QtGui, QtCore
from regDecoder import Ui_Form
import sys

#ref: https://doc.qt.io/qt-5/qtablewidgetitem.html
#ref: https://www.regexpal.com/93640

def set_bit(value, bit):
    return value | (1 << bit)

def clear_bit(value, bit):
    return value & ~(1 << bit)

class MainWindow(QtWidgets.QMainWindow):
    def __init__(self):
        super(MainWindow, self).__init__()
        self.ui = Ui_Form()
        self.ui.setupUi(self)
        self.ui.pushButton.clicked.connect(self.buttonClicked)
        self.ui.pushButtonCleanSelection.clicked.connect(self.buttonCleanSelection)
        self.ui.tableWidget.itemDoubleClicked.connect(self.itemDoubleClickedToInverse)
        self.ui.tableWidget.itemSelectionChanged.connect(self.itemSelectionChangedSlot)
        self.ui.tableWidget.horizontalHeader().setSectionResizeMode(QtWidgets.QHeaderView.Stretch)
        self.inputIntValue = 0xFF

    def itemSelectionChangedSlot(self):
        # print('itemSelectionChangedSlot is ')
        indexList = []
        for item in self.ui.tableWidget.selectedItems():
            indexList.append([31 - item.column(), item.text()])
        if indexList == []:  # select nothing, take it as select all
            for columnIndex in range(0, 32):
                indexList.append([31 - columnIndex, self.ui.tableWidget.item(0, columnIndex).text()])
        indexList.sort()  # indexList.sort(reverse=True)
        # print(indexList)

        selectedValueText = ""
        for list in indexList:
            selectedValueText = list[1] + selectedValueText
        # print(selectedValueText)
        # show lineEditOutXXX
        self.ui.lineEditOutBin.setText("0b" + selectedValueText)
        self.ui.lineEditOutDec.setText(str(int(selectedValueText, base=2)))
        self.ui.lineEditOutHex.setText(hex(int(selectedValueText, base=2)).upper().replace("0X", "0x"))
        # show labelOutputResult
        indexList.sort(reverse=True)
        selectedIndexText = []
        for list in indexList:
            # print(str(list[0]))
            selectedIndexText.append(str(list[0]))
        if len(selectedIndexText) == 32:
            self.ui.labelOutputResult.setText("Output Result: ")
        else:
            self.ui.labelOutputResult.setText("partial Selection: [" + ', '.join(selectedIndexText) + "]")

    def itemDoubleClickedToInverse(self, item):
        print('item double click. item index is ' + str(31 - item.column()))
        itemIndex = 31 - item.column()
        if item.text() == "0":
            item.setText("1")
            self.inputIntValue = set_bit(self.inputIntValue, itemIndex)
        else:
            item.setText("0")
            self.inputIntValue = clear_bit(self.inputIntValue, itemIndex)
        self.buttonCleanSelection()

    def buttonCleanSelection(self):
        print('buttonCleanSelection: ')
        for item in self.ui.tableWidget.selectedItems():
            item.setSelected(False)

    def buttonClicked(self):
        text = self.ui.lineEdit.text()
        # ref https://vimsky.com/zh-tw/examples/detail/python-method-PyQt4.QtGui.QFont.html
        palettePass = QtGui.QPalette()
        palettePass.setColor(QtGui.QPalette.Text, QtCore.Qt.black)
        fontPass = QtGui.QFont()
        fontPass.setPointSize(12)
        fontPass.setBold(False)
        paletteFail = QtGui.QPalette()
        paletteFail.setColor(QtGui.QPalette.Text, QtCore.Qt.red)
        fontFail = QtGui.QFont()
        fontFail.setPointSize(16)
        fontFail.setBold(True)
        print("buttonClicked: text is " + str(text))
        try:
            if text[0:2] == "0x":  # if hex
                intVal = int(text[2:], 16)
                binVal = bin(intVal)
            elif text[0:2] == "0b":  # if bin
                intVal = int(text[2:], 2)
                binVal = bin(intVal)
            else:  # if dec
                intVal = int(text, 10)
                binVal = bin(intVal)
            #print(" binVal is " + binVal)
            #print(" intVal is " + str(intVal))
            if intVal > (pow(2,32)-1):
                intVal = (pow(2,32)-1)
                binVal = bin(intVal)
                self.ui.lineEdit.setPalette(paletteFail)
                self.ui.lineEdit.setFont(fontFail)
                self.ui.labelInputCheckResult.setText(str(text) + ' is invalid input. Take it as 0xFFFFFFFF.  \nNOTE: Only Support 0~2^32-1.')
            elif intVal < 0:
                intVal = 0
                binVal = bin(intVal)
                self.ui.lineEdit.setPalette(paletteFail)
                self.ui.lineEdit.setFont(fontFail)
                self.ui.labelInputCheckResult.setText(str(text)+' is invalid input. Take it as 0. \nNOTE: Only Support 0~2^32-1.')
            else:
                self.ui.lineEdit.setPalette(palettePass)
                self.ui.lineEdit.setFont(fontPass)
                self.ui.labelInputCheckResult.setText("Input Check Result: PASS.")
                print("Input Check Result: PASS.")
            self.inputIntValue = intVal
        except:
            intVal = self.inputIntValue
            binVal = bin(intVal)
            self.inputIntValue = self.inputIntValue
            self.ui.lineEdit.setPalette(paletteFail)
            self.ui.lineEdit.setFont(fontFail)
            self.ui.labelInputCheckResult.setText(str(text)+' is invalid input. \nNOTE: "^0x[0-9A-Fa-f]+$" for hex. "^0b[0-1]+$" for binary. "^[0-9]+$" for decimal.')
                 #    /^       # start of string
                 #    0x       # 0x prefix
                 #    [0-9A-F] # a hex digit
                 #    $        # end of string
        # for used:
        index = -1
        while binVal[index] in ["1", "0"]:
            item = self.ui.tableWidget.item(0, 32 + index)
            item.setText(binVal[index])
            index = index - 1
        # for not used: make all zero
        while 32 + index >= 0:
            item = self.ui.tableWidget.item(0, 32 + index)
            item.setText("0")
            index = index - 1
        self.buttonCleanSelection()
        self.itemSelectionChangedSlot()

    def keyPressEvent(self, qKeyEvent):
        #print(qKeyEvent.key())
        if qKeyEvent.key() == QtCore.Qt.Key_Return or qKeyEvent.key() == QtCore.Qt.Key_Enter:
            self.buttonClicked()
            print('Enter pressed: ')
        else:
            super().keyPressEvent(qKeyEvent)


if __name__ == '__main__':
    app = QtWidgets.QApplication([])
    window = MainWindow()
    window.setWindowTitle("regDecodec ver.20200607")

    brush = QtGui.QBrush(QtGui.QColor(245, 245, 245))
    brush.setStyle(QtCore.Qt.SolidPattern)
    for i in [0,16] :
        for j in range(0,8) :
            print(i+j)
            window.ui.tableWidget.item(0, i+j).setBackground(brush)


    window.show()
    sys.exit(app.exec_())



```

### regDecoder.py
```
# -*- coding: utf-8 -*-

# Form implementation generated from reading ui file 'regDecoder.ui'
#
# Created by: PyQt5 UI code generator 5.15.0
#
# WARNING: Any manual changes made to this file will be lost when pyuic5 is
# run again.  Do not edit this file unless you know what you are doing.


from PyQt5 import QtCore, QtGui, QtWidgets


class Ui_Form(object):
    def setupUi(self, Form):
        Form.setObjectName("Form")
        Form.resize(1043, 275)
        Form.setMinimumSize(QtCore.QSize(1043, 275))
        Form.setMaximumSize(QtCore.QSize(1043, 275))
        self.tableWidget = QtWidgets.QTableWidget(Form)
        self.tableWidget.setGeometry(QtCore.QRect(10, 100, 1021, 61))
        self.tableWidget.setEditTriggers(QtWidgets.QAbstractItemView.NoEditTriggers)
        self.tableWidget.setObjectName("tableWidget")
        self.tableWidget.setColumnCount(32)
        self.tableWidget.setRowCount(1)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setVerticalHeaderItem(0, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(0, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(1, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(2, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(3, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(4, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(5, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(6, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(7, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(8, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(9, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(10, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(11, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(12, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(13, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(14, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(15, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(16, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(17, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(18, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(19, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(20, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(21, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(22, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(23, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(24, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(25, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(26, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(27, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(28, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(29, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(30, item)
        item = QtWidgets.QTableWidgetItem()
        self.tableWidget.setHorizontalHeaderItem(31, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        brush = QtGui.QBrush(QtGui.QColor(255, 255, 255))
        brush.setStyle(QtCore.Qt.NoBrush)
        item.setBackground(brush)
        self.tableWidget.setItem(0, 0, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        brush = QtGui.QBrush(QtGui.QColor(255, 255, 255))
        brush.setStyle(QtCore.Qt.NoBrush)
        item.setBackground(brush)
        self.tableWidget.setItem(0, 1, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        brush = QtGui.QBrush(QtGui.QColor(211, 211, 211))
        brush.setStyle(QtCore.Qt.NoBrush)
        item.setBackground(brush)
        self.tableWidget.setItem(0, 2, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        brush = QtGui.QBrush(QtGui.QColor(211, 211, 211))
        brush.setStyle(QtCore.Qt.NoBrush)
        item.setBackground(brush)
        self.tableWidget.setItem(0, 3, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        brush = QtGui.QBrush(QtGui.QColor(0, 0, 0))
        brush.setStyle(QtCore.Qt.NoBrush)
        item.setForeground(brush)
        self.tableWidget.setItem(0, 4, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 5, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 6, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 7, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 8, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 9, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 10, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 11, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 12, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 13, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 14, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 15, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 16, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 17, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 18, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 19, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 20, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 21, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 22, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 23, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 24, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 25, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 26, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 27, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 28, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 29, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 30, item)
        item = QtWidgets.QTableWidgetItem()
        item.setTextAlignment(QtCore.Qt.AlignCenter)
        self.tableWidget.setItem(0, 31, item)
        self.tableWidget.verticalHeader().setVisible(True)
        self.tableWidget.verticalHeader().setCascadingSectionResizes(True)
        self.tableWidget.verticalHeader().setStretchLastSection(True)
        self.lineEdit = QtWidgets.QLineEdit(Form)
        self.lineEdit.setGeometry(QtCore.QRect(30, 20, 301, 41))
        font = QtGui.QFont()
        font.setPointSize(12)
        self.lineEdit.setFont(font)
        self.lineEdit.setObjectName("lineEdit")
        self.pushButton = QtWidgets.QPushButton(Form)
        self.pushButton.setGeometry(QtCore.QRect(340, 20, 131, 41))
        font = QtGui.QFont()
        font.setPointSize(12)
        font.setBold(False)
        font.setWeight(50)
        self.pushButton.setFont(font)
        self.pushButton.setObjectName("pushButton")
        self.labelInputCheckResult = QtWidgets.QLabel(Form)
        self.labelInputCheckResult.setGeometry(QtCore.QRect(480, 20, 551, 41))
        font = QtGui.QFont()
        font.setPointSize(12)
        self.labelInputCheckResult.setFont(font)
        self.labelInputCheckResult.setObjectName("labelInputCheckResult")
        self.labelOutputResult = QtWidgets.QLabel(Form)
        self.labelOutputResult.setEnabled(True)
        self.labelOutputResult.setGeometry(QtCore.QRect(70, 180, 961, 31))
        font = QtGui.QFont()
        font.setPointSize(12)
        self.labelOutputResult.setFont(font)
        self.labelOutputResult.setAlignment(QtCore.Qt.AlignRight|QtCore.Qt.AlignTrailing|QtCore.Qt.AlignVCenter)
        self.labelOutputResult.setObjectName("labelOutputResult")
        self.pushButtonCleanSelection = QtWidgets.QPushButton(Form)
        self.pushButtonCleanSelection.setGeometry(QtCore.QRect(930, 70, 101, 23))
        self.pushButtonCleanSelection.setObjectName("pushButtonCleanSelection")
        self.widget = QtWidgets.QWidget(Form)
        self.widget.setGeometry(QtCore.QRect(480, 210, 551, 51))
        self.widget.setObjectName("widget")
        self.horizontalLayout = QtWidgets.QHBoxLayout(self.widget)
        self.horizontalLayout.setContentsMargins(0, 0, 0, 0)
        self.horizontalLayout.setObjectName("horizontalLayout")
        self.lineEditOutBin = QtWidgets.QLineEdit(self.widget)
        sizePolicy = QtWidgets.QSizePolicy(QtWidgets.QSizePolicy.Expanding, QtWidgets.QSizePolicy.Expanding)
        sizePolicy.setHorizontalStretch(3)
        sizePolicy.setVerticalStretch(0)
        sizePolicy.setHeightForWidth(self.lineEditOutBin.sizePolicy().hasHeightForWidth())
        self.lineEditOutBin.setSizePolicy(sizePolicy)
        font = QtGui.QFont()
        font.setPointSize(12)
        self.lineEditOutBin.setFont(font)
        self.lineEditOutBin.setReadOnly(True)
        self.lineEditOutBin.setObjectName("lineEditOutBin")
        self.horizontalLayout.addWidget(self.lineEditOutBin)
        self.lineEditOutDec = QtWidgets.QLineEdit(self.widget)
        sizePolicy = QtWidgets.QSizePolicy(QtWidgets.QSizePolicy.Expanding, QtWidgets.QSizePolicy.Expanding)
        sizePolicy.setHorizontalStretch(1)
        sizePolicy.setVerticalStretch(0)
        sizePolicy.setHeightForWidth(self.lineEditOutDec.sizePolicy().hasHeightForWidth())
        self.lineEditOutDec.setSizePolicy(sizePolicy)
        font = QtGui.QFont()
        font.setPointSize(12)
        self.lineEditOutDec.setFont(font)
        self.lineEditOutDec.setReadOnly(True)
        self.lineEditOutDec.setObjectName("lineEditOutDec")
        self.horizontalLayout.addWidget(self.lineEditOutDec)
        self.lineEditOutHex = QtWidgets.QLineEdit(self.widget)
        sizePolicy = QtWidgets.QSizePolicy(QtWidgets.QSizePolicy.Expanding, QtWidgets.QSizePolicy.Expanding)
        sizePolicy.setHorizontalStretch(1)
        sizePolicy.setVerticalStretch(0)
        sizePolicy.setHeightForWidth(self.lineEditOutHex.sizePolicy().hasHeightForWidth())
        self.lineEditOutHex.setSizePolicy(sizePolicy)
        font = QtGui.QFont()
        font.setPointSize(12)
        self.lineEditOutHex.setFont(font)
        self.lineEditOutHex.setReadOnly(True)
        self.lineEditOutHex.setObjectName("lineEditOutHex")
        self.horizontalLayout.addWidget(self.lineEditOutHex)

        self.retranslateUi(Form)
        QtCore.QMetaObject.connectSlotsByName(Form)

    def retranslateUi(self, Form):
        _translate = QtCore.QCoreApplication.translate
        Form.setWindowTitle(_translate("Form", "Form"))
        item = self.tableWidget.verticalHeaderItem(0)
        item.setText(_translate("Form", "value"))
        item = self.tableWidget.horizontalHeaderItem(0)
        item.setText(_translate("Form", "31"))
        item = self.tableWidget.horizontalHeaderItem(1)
        item.setText(_translate("Form", "30"))
        item = self.tableWidget.horizontalHeaderItem(2)
        item.setText(_translate("Form", "29"))
        item = self.tableWidget.horizontalHeaderItem(3)
        item.setText(_translate("Form", "28"))
        item = self.tableWidget.horizontalHeaderItem(4)
        item.setText(_translate("Form", "27"))
        item = self.tableWidget.horizontalHeaderItem(5)
        item.setText(_translate("Form", "26"))
        item = self.tableWidget.horizontalHeaderItem(6)
        item.setText(_translate("Form", "25"))
        item = self.tableWidget.horizontalHeaderItem(7)
        item.setText(_translate("Form", "24"))
        item = self.tableWidget.horizontalHeaderItem(8)
        item.setText(_translate("Form", "23"))
        item = self.tableWidget.horizontalHeaderItem(9)
        item.setText(_translate("Form", "22"))
        item = self.tableWidget.horizontalHeaderItem(10)
        item.setText(_translate("Form", "21"))
        item = self.tableWidget.horizontalHeaderItem(11)
        item.setText(_translate("Form", "20"))
        item = self.tableWidget.horizontalHeaderItem(12)
        item.setText(_translate("Form", "19"))
        item = self.tableWidget.horizontalHeaderItem(13)
        item.setText(_translate("Form", "18"))
        item = self.tableWidget.horizontalHeaderItem(14)
        item.setText(_translate("Form", "17"))
        item = self.tableWidget.horizontalHeaderItem(15)
        item.setText(_translate("Form", "16"))
        item = self.tableWidget.horizontalHeaderItem(16)
        item.setText(_translate("Form", "15"))
        item = self.tableWidget.horizontalHeaderItem(17)
        item.setText(_translate("Form", "14"))
        item = self.tableWidget.horizontalHeaderItem(18)
        item.setText(_translate("Form", "13"))
        item = self.tableWidget.horizontalHeaderItem(19)
        item.setText(_translate("Form", "12"))
        item = self.tableWidget.horizontalHeaderItem(20)
        item.setText(_translate("Form", "11"))
        item = self.tableWidget.horizontalHeaderItem(21)
        item.setText(_translate("Form", "10"))
        item = self.tableWidget.horizontalHeaderItem(22)
        item.setText(_translate("Form", "9"))
        item = self.tableWidget.horizontalHeaderItem(23)
        item.setText(_translate("Form", "8"))
        item = self.tableWidget.horizontalHeaderItem(24)
        item.setText(_translate("Form", "7"))
        item = self.tableWidget.horizontalHeaderItem(25)
        item.setText(_translate("Form", "6"))
        item = self.tableWidget.horizontalHeaderItem(26)
        item.setText(_translate("Form", "5"))
        item = self.tableWidget.horizontalHeaderItem(27)
        item.setText(_translate("Form", "4"))
        item = self.tableWidget.horizontalHeaderItem(28)
        item.setText(_translate("Form", "3"))
        item = self.tableWidget.horizontalHeaderItem(29)
        item.setText(_translate("Form", "2"))
        item = self.tableWidget.horizontalHeaderItem(30)
        item.setText(_translate("Form", "1"))
        item = self.tableWidget.horizontalHeaderItem(31)
        item.setText(_translate("Form", "0"))
        __sortingEnabled = self.tableWidget.isSortingEnabled()
        self.tableWidget.setSortingEnabled(False)
        item = self.tableWidget.item(0, 0)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 1)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 2)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 3)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 4)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 5)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 6)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 7)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 8)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 9)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 10)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 11)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 12)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 13)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 14)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 15)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 16)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 17)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 18)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 19)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 20)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 21)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 22)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 23)
        item.setText(_translate("Form", "0"))
        item = self.tableWidget.item(0, 24)
        item.setText(_translate("Form", "1"))
        item = self.tableWidget.item(0, 25)
        item.setText(_translate("Form", "1"))
        item = self.tableWidget.item(0, 26)
        item.setText(_translate("Form", "1"))
        item = self.tableWidget.item(0, 27)
        item.setText(_translate("Form", "1"))
        item = self.tableWidget.item(0, 28)
        item.setText(_translate("Form", "1"))
        item = self.tableWidget.item(0, 29)
        item.setText(_translate("Form", "1"))
        item = self.tableWidget.item(0, 30)
        item.setText(_translate("Form", "1"))
        item = self.tableWidget.item(0, 31)
        item.setText(_translate("Form", "1"))
        self.tableWidget.setSortingEnabled(__sortingEnabled)
        self.lineEdit.setText(_translate("Form", "0xFF"))
        self.pushButton.setText(_translate("Form", "Show it"))
        self.labelInputCheckResult.setText(_translate("Form", "Only Support 0~2^32-1. NOTE: \"^0x[0-9A-Fa-f]+$\" for hex. \"^0b[0-1]+$\" for binary."))
        self.labelOutputResult.setText(_translate("Form", "Output Result: "))
        self.pushButtonCleanSelection.setText(_translate("Form", "clean selection"))
        self.lineEditOutBin.setText(_translate("Form", "0b11111111"))
        self.lineEditOutDec.setText(_translate("Form", "255"))
        self.lineEditOutHex.setText(_translate("Form", "0xFF"))

```
### regDecoder.ui
```
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>Form</class>
 <widget class="QWidget" name="Form">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>1043</width>
    <height>275</height>
   </rect>
  </property>
  <property name="minimumSize">
   <size>
    <width>1043</width>
    <height>275</height>
   </size>
  </property>
  <property name="maximumSize">
   <size>
    <width>1043</width>
    <height>275</height>
   </size>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <widget class="QTableWidget" name="tableWidget">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>100</y>
     <width>1021</width>
     <height>61</height>
    </rect>
   </property>
   <property name="editTriggers">
    <set>QAbstractItemView::NoEditTriggers</set>
   </property>
   <attribute name="verticalHeaderVisible">
    <bool>true</bool>
   </attribute>
   <attribute name="verticalHeaderCascadingSectionResizes">
    <bool>true</bool>
   </attribute>
   <attribute name="verticalHeaderStretchLastSection">
    <bool>true</bool>
   </attribute>
   <row>
    <property name="text">
     <string>value</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </row>
   <column>
    <property name="text">
     <string>31</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>30</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>29</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>28</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>27</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>26</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>25</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>24</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>23</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>22</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>21</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>20</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>19</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>18</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>17</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>16</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>15</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>14</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>13</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>12</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>11</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>10</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>9</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>8</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>7</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>6</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>5</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>4</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>3</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>2</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>1</string>
    </property>
   </column>
   <column>
    <property name="text">
     <string>0</string>
    </property>
   </column>
   <item row="0" column="0">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
    <property name="background">
     <brush brushstyle="NoBrush">
      <color alpha="255">
       <red>255</red>
       <green>255</green>
       <blue>255</blue>
      </color>
     </brush>
    </property>
   </item>
   <item row="0" column="1">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
    <property name="background">
     <brush brushstyle="NoBrush">
      <color alpha="255">
       <red>255</red>
       <green>255</green>
       <blue>255</blue>
      </color>
     </brush>
    </property>
   </item>
   <item row="0" column="2">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
    <property name="background">
     <brush brushstyle="NoBrush">
      <color alpha="255">
       <red>211</red>
       <green>211</green>
       <blue>211</blue>
      </color>
     </brush>
    </property>
   </item>
   <item row="0" column="3">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
    <property name="background">
     <brush brushstyle="NoBrush">
      <color alpha="255">
       <red>211</red>
       <green>211</green>
       <blue>211</blue>
      </color>
     </brush>
    </property>
   </item>
   <item row="0" column="4">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
    <property name="foreground">
     <brush brushstyle="NoBrush">
      <color alpha="255">
       <red>0</red>
       <green>0</green>
       <blue>0</blue>
      </color>
     </brush>
    </property>
   </item>
   <item row="0" column="5">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="6">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="7">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="8">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="9">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="10">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="11">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="12">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="13">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="14">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="15">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="16">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="17">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="18">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="19">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="20">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="21">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="22">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="23">
    <property name="text">
     <string>0</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="24">
    <property name="text">
     <string>1</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="25">
    <property name="text">
     <string>1</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="26">
    <property name="text">
     <string>1</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="27">
    <property name="text">
     <string>1</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="28">
    <property name="text">
     <string>1</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="29">
    <property name="text">
     <string>1</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="30">
    <property name="text">
     <string>1</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
   <item row="0" column="31">
    <property name="text">
     <string>1</string>
    </property>
    <property name="textAlignment">
     <set>AlignCenter</set>
    </property>
   </item>
  </widget>
  <widget class="QLineEdit" name="lineEdit">
   <property name="geometry">
    <rect>
     <x>30</x>
     <y>20</y>
     <width>301</width>
     <height>41</height>
    </rect>
   </property>
   <property name="font">
    <font>
     <pointsize>12</pointsize>
    </font>
   </property>
   <property name="text">
    <string>0xFF</string>
   </property>
  </widget>
  <widget class="QPushButton" name="pushButton">
   <property name="geometry">
    <rect>
     <x>340</x>
     <y>20</y>
     <width>131</width>
     <height>41</height>
    </rect>
   </property>
   <property name="font">
    <font>
     <pointsize>12</pointsize>
     <weight>50</weight>
     <bold>false</bold>
    </font>
   </property>
   <property name="text">
    <string>Show it</string>
   </property>
  </widget>
  <widget class="QLabel" name="labelInputCheckResult">
   <property name="geometry">
    <rect>
     <x>480</x>
     <y>20</y>
     <width>551</width>
     <height>41</height>
    </rect>
   </property>
   <property name="font">
    <font>
     <pointsize>12</pointsize>
    </font>
   </property>
   <property name="text">
    <string>Only Support 0~2^32-1. NOTE: &quot;^0x[0-9A-Fa-f]+$&quot; for hex. &quot;^0b[0-1]+$&quot; for binary.</string>
   </property>
  </widget>
  <widget class="QLabel" name="labelOutputResult">
   <property name="enabled">
    <bool>true</bool>
   </property>
   <property name="geometry">
    <rect>
     <x>70</x>
     <y>180</y>
     <width>961</width>
     <height>31</height>
    </rect>
   </property>
   <property name="font">
    <font>
     <pointsize>12</pointsize>
    </font>
   </property>
   <property name="text">
    <string>Output Result: </string>
   </property>
   <property name="alignment">
    <set>Qt::AlignRight|Qt::AlignTrailing|Qt::AlignVCenter</set>
   </property>
  </widget>
  <widget class="QPushButton" name="pushButtonCleanSelection">
   <property name="geometry">
    <rect>
     <x>930</x>
     <y>70</y>
     <width>101</width>
     <height>23</height>
    </rect>
   </property>
   <property name="text">
    <string>clean selection</string>
   </property>
  </widget>
  <widget class="QWidget" name="">
   <property name="geometry">
    <rect>
     <x>480</x>
     <y>210</y>
     <width>551</width>
     <height>51</height>
    </rect>
   </property>
   <layout class="QHBoxLayout" name="horizontalLayout">
    <item>
     <widget class="QLineEdit" name="lineEditOutBin">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
        <horstretch>3</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="font">
       <font>
        <pointsize>12</pointsize>
       </font>
      </property>
      <property name="text">
       <string>0b11111111</string>
      </property>
      <property name="readOnly">
       <bool>true</bool>
      </property>
     </widget>
    </item>
    <item>
     <widget class="QLineEdit" name="lineEditOutDec">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
        <horstretch>1</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="font">
       <font>
        <pointsize>12</pointsize>
       </font>
      </property>
      <property name="text">
       <string>255</string>
      </property>
      <property name="readOnly">
       <bool>true</bool>
      </property>
     </widget>
    </item>
    <item>
     <widget class="QLineEdit" name="lineEditOutHex">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
        <horstretch>1</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="font">
       <font>
        <pointsize>12</pointsize>
       </font>
      </property>
      <property name="text">
       <string>0xFF</string>
      </property>
      <property name="readOnly">
       <bool>true</bool>
      </property>
     </widget>
    </item>
   </layout>
  </widget>
 </widget>
 <resources/>
 <connections/>
</ui>

```
