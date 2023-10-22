from PyQt5.QtWidgets import QApplication, QWidget, QPushButton, QLabel, QListWidget, QLineEdit, QTextEdit, QInputDialog, \
    QHBoxLayout, QVBoxLayout, QFormLayout

import json

try:
    with open("notes_data.json", "r", encoding="utf-8") as file:
        notes = json.load(file)
except:
    notes = {}


app = QApplication([])

app.setStyleSheet("""
     QWidget{
          background-color: #A0E7E5;
          }
     QPushButton{

        background-color: #BCECE0;
        border-width: 10px;
        border-color: #04ECF0;
        border-style: inset;
        font-family: Impact;
        min-width: 0em;
        padding: 10px;
        border-radius: 10px;

        }
     QGroupBox {
        border-style: inset;
        border-width: 10px;
        border-color: #059DC0;
        border-radius: 10px;
        font-family: Impact;
        }
     QSpinBox {
        border-style: inset;
        border-width: 10px;
        border-color: #15B5B0;
        border-radius: 10px;
        font-family: Impact;
        }
        QLabel {
        border-style: inset;
        border-width: 10px;
        max-height: 20px;
        man-width: 40px;
        border-color: #E7F2F8;
        border-radius: 10px;
        font-family: Impact;
        }
          """)
notes_win = QWidget()
notes_win.setWindowTitle('Розумні замітки')
notes_win.resize(900, 600)

# віджети вікна програми
list_notes = QListWidget()
list_notes_label = QLabel('Список заміток')

button_note_create = QPushButton('Створити замітку')  # з'являється вікно з полем "Введіть ім'я замітки"
button_note_del = QPushButton('Видалити замітку')
button_note_save = QPushButton('Зберегти замітку')

field_tag = QLineEdit('')
field_tag.setPlaceholderText('Введіть тег...')
field_text = QTextEdit()
button_tag_add = QPushButton('Додати до замітки')
button_tag_del = QPushButton('Відкріпити від замітки')
button_tag_search = QPushButton('Шукати замітки за тегом')
list_tags = QListWidget()
list_tags_label = QLabel('Список тегів')

# розташування віджетів по лейаутах
layout_notes = QHBoxLayout()
col_1 = QVBoxLayout()
col_1.addWidget(field_text)

col_2 = QVBoxLayout()
col_2.addWidget(list_notes_label)
col_2.addWidget(list_notes)
row_1 = QHBoxLayout()
row_1.addWidget(button_note_create)
row_1.addWidget(button_note_del)
row_2 = QHBoxLayout()
row_2.addWidget(button_note_save)
col_2.addLayout(row_1)
col_2.addLayout(row_2)

col_2.addWidget(list_tags_label)
col_2.addWidget(list_tags)
col_2.addWidget(field_tag)
row_3 = QHBoxLayout()
row_3.addWidget(button_tag_add)
row_3.addWidget(button_tag_del)
row_4 = QHBoxLayout()
row_4.addWidget(button_tag_search)

col_2.addLayout(row_3)
col_2.addLayout(row_4)

layout_notes.addLayout(col_1, stretch=2)
layout_notes.addLayout(col_2, stretch=1)
notes_win.setLayout(layout_notes)

'''Функціонал програми'''

'''Робота з текстом замітки'''


def add_note():
    note_name, ok = QInputDialog.getText(notes_win, "Додати замітку", "Назва замітки: ")
    if ok and note_name != "":
        notes[note_name] = {
            "текст": "",
            "теги": []
        }
        list_notes.clear()
        field_text.clear()
        list_notes.addItems(notes)
        with open("notes_data.json", "w", encoding="utf-8") as file:
            json.dump(notes, file, ensure_ascii=False, indent=4)










def show_note():
    # отримуємо текст із замітки з виділеною назвою та відображаємо її в полі редагування
    key = list_notes.selectedItems()[0].text()
    print(key)
    field_text.setText(notes[key]["текст"])
    list_tags.clear()
    list_tags.addItems(notes[key]["теги"])

list_notes.itemClicked.connect(show_note)

def save_note():
    if list_notes.selectedItems():
        key = list_notes.selectedItems()[0].text()
        notes[key]["текст"] = field_text.toPlainText()
        with open("notes_data.json", "w", encoding="utf-8") as file:
            json.dump(notes, file, ensure_ascii=False, indent=4)
    else:
        print("Замітка для збереження не вибрана!")


def del_note():
    if list_notes.selectedItems():
        key = list_notes.selectedItems()[0].text()
        notes.pop(key)
        list_notes.clear()
        list_tags.clear()
        field_text.clear()
        list_notes.addItems(notes)
        with open("notes_data.json", "w", encoding="utf-8") as file:
            json.dump(notes, file, sort_keys=True, ensure_ascii=False, indent=4)
        print(notes)
    else:
        print("Замітка для вилучення не обрана!")


'''Работа з тегами замітки'''


def add_tag():#кнопка добавити тег
    if list_notes.selectedItems():
        key = list_notes.selectedItems()[0].text()
        tag = field_tag.text()
        if not tag in notes[key]["теги"]:
            notes[key]["теги"].append(tag)
            list_tags.addItem(tag)
            field_tag.clear()
        with open("notes_data.json", "w", encoding="utf-8") as file:
            json.dump(notes, file,  ensure_ascii=False)
        print(notes)
    else:
        print("Замітка для додавання тега не обрана!")


def del_tag(): #кнопка видалити тег
    if list_tags.selectedItems():
        key = list_notes.selectedItems()[0].text()
        tag = list_tags.selectedItems()[0].text()
        notes[key]["теги"].remove(tag)
        list_tags.clear()
        list_tags.addItems(notes[key]["теги"])
        with open("notes_data.json", "w", encoding="utf-8") as file:
            json.dump(notes, file, ensure_ascii=False)
    else:
        print("Тег для вилучення не обраний!")


def search_tag(): #кнопка "шукати замітку за тегом"
    button_text = button_tag_search.text()
    tag = field_tag.text()

    if button_text == "Шукати замітки за тегом":
        apply_tag_search(tag)
    elif button_text == "Скинути пошук":
        reset_search()

def apply_tag_search(tag):
    notes_filtered = {}
    for note, value in notes.items():
        if tag in value["теги"]:
            notes_filtered[note] = value

    button_tag_search.setText("Скинути пошук")
    list_notes.clear()
    list_tags.clear()
    list_notes.addItems(notes_filtered)

def reset_search():
    field_tag.clear()
    list_notes.clear()
    list_tags.clear()
    list_notes.addItems(notes)
    button_tag_search.setText("Шукати замітки за тегом")



'''Запуск програми'''
# підключення обробки подій
button_note_create.clicked.connect(add_note)

button_note_save.clicked.connect(save_note)
button_note_del.clicked.connect(del_note)
button_tag_add.clicked.connect(add_tag)
button_tag_del.clicked.connect(del_tag)
button_tag_search.clicked.connect(search_tag)

# запуск програми
notes_win.show()


list_notes.addItems(notes)

app.exec_()
