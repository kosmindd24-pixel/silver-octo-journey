import json
from datetime import datetime

# ==================== Модуль 1: Класс Entry ====================
class Entry:
    """Класс для представления одной записи в дневнике."""
    def __init__(self, date, description, hours):
        # Проверка и форматирование даты
        try:
            datetime.strptime(date, "%Y-%m-%d")
        except ValueError:
            raise ValueError("Неверный формат даты. Используйте ГГГГ-ММ-ДД")
        self.date = date
        self.description = description
        self.hours = hours

    def to_dict(self):
        """Преобразует объект Entry в словарь для сохранения в JSON."""
        return {
            "date": self.date,
            "description": self.description,
            "hours": self.hours
        }

    @staticmethod
    def from_dict(data):
        """Создает объект Entry из словаря (при загрузке из JSON)."""
        return Entry(data["date"], data["description"], data["hours"])


# ==================== Модуль 2: Класс Diary ====================
class Diary:
    """Главный класс для управления всеми записями дневника."""
    def __init__(self, filename="diary.json"):
        self.filename = filename
        self.entries = []
        self.load()  # Автоматически загружаем данные при создании объекта

    def add_entry(self, date, description, hours):
        """Добавляет новую запись."""
        try:
            new_entry = Entry(date, description, hours)
            self.entries.append(new_entry)
            self.save()  # Сразу сохраняем после добавления
            print("✓ Запись успешно добавлена!")
        except ValueError as e:
            print(f"Ошибка: {e}")

    def list_entries(self):
        """Выводит все записи в отформатированном виде."""
        if not self.entries:
            print("Дневник пуст.")
            return
        print("\n" + "=" * 60)
        for i, entry in enumerate(self.entries, start=1):
            print(f"{i}. {entry.date} | {entry.hours} ч. | {entry.description}")
        print("=" * 60)

    def edit_entry(self, index, new_date, new_description, new_hours):
        """Редактирует запись по индексу."""
        if not (0 <= index < len(self.entries)):
            print("Ошибка: Запись с таким номером не найдена.")
            return
        try:
            # Создаем временную запись для валидации данных
            edited_entry = Entry(new_date, new_description, new_hours)
            self.entries[index] = edited_entry
            self.save()
            print("✓ Запись успешно отредактирована!")
        except ValueError as e:
            print(f"Ошибка: {e}")

    def delete_entry(self, index):
        """Удаляет запись по индексу."""
        if not (0 <= index < len(self.entries)):
            print("Ошибка: Запись с таким номером не найдена.")
            return
        deleted = self.entries.pop(index)
        self.save()
        print(f"✓ Запись '{deleted.description}' успешно удалена.")

    def get_total_hours(self):
        """Подсчитывает и возвращает общее количество часов."""
        total = sum(entry.hours for entry in self.entries)
        return total

    def search_entries(self, keyword):
        """Ищет записи, содержащие ключевое слово в описании."""
        found = [entry for entry in self.entries if keyword.lower() in entry.description.lower()]
        if not found:
            print("Записи по вашему запросу не найдены.")
            return
        print(f"\nРезультаты поиска по слову '{keyword}':")
        print("=" * 60)
        for entry in found:
            print(f"{entry.date} | {entry.hours} ч. | {entry.description}")
        print("=" * 60)

    def save(self):
        """Сохраняет все записи в JSON-файл."""
        data = [entry.to_dict() for entry in self.entries]
        with open(self.filename, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False, indent=4)

    def load(self):
        """Загружает записи из JSON-файла при запуске."""
        try:
            with open(self.filename, 'r', encoding='utf-8') as f:
                data = json.load(f)
                self.entries = [Entry.from_dict(item) for item in data]
        except FileNotFoundError:
            # Если файла нет, просто начинаем с пустого списка
            self.entries = []
        except json.JSONDecodeError:
            print("Предупреждение: Файл поврежден. Начинаем с пустого дневника.")
            self.entries = []


# ==================== Модуль 3: Пользовательский интерфейс ====================
def main():
    diary = Diary()

    while True:
        print("\n" + "=" * 40)
        print("     ДНЕВНИК ПРОИЗВОДСТВЕННОЙ ПРАКТИКИ")
        print("=" * 40)
        print("1. Добавить запись")
        print("2. Показать все записи")
        print("3. Редактировать запись")
        print("4. Удалить запись")
        print("5. Показать итоговое количество часов")
        print("6. Поиск по ключевому слову")
        print("0. Выход")
        print("=" * 40)

        choice = input("Выберите действие: ")

        if choice == '1':
            date = input("Дата (ГГГГ-ММ-ДД): ")
            description = input("Описание задачи: ")
            try:
                hours = int(input("Количество часов: "))
                diary.add_entry(date, description, hours)
            except ValueError:
                print("Ошибка: Часы должны быть числом.")

        elif choice == '2':
            diary.list_entries()

        elif choice == '3':
            diary.list_entries()
            try:
                index = int(input("Введите номер записи для редактирования: ")) - 1
                if 0 <= index < len(diary.entries):
                    new_date = input(f"Новая дата ({diary.entries[index].date}): ") or diary.entries[index].date
                    new_description = input(f"Новое описание ({diary.entries[index].description}): ") or diary.entries[index].description
                    new_hours_str = input(f"Новое количество часов ({diary.entries[index].hours}): ")
                    new_hours = int(new_hours_str) if new_hours_str else diary.entries[index].hours
                    diary.edit_entry(index, new_date, new_description, new_hours)
                else:
                    print("Неверный номер.")
            except ValueError:
                print("Неверный ввод.")

        elif choice == '4':
            diary.list_entries()
            try:
                index = int(input("Введите номер записи для удаления: ")) - 1
                diary.delete_entry(index)
            except ValueError:
                print("Неверный ввод.")

        elif choice == '5':
            total = diary.get_total_hours()
            print(f"\nОбщее количество отработанных часов: {total}")

        elif choice == '6':
            keyword = input("Введите ключевое слово для поиска: ")
            diary.search_entries(keyword)

        elif choice == '0':
            print("Выход из программы. До свидания!")
            break
        else:
            print("Неверный пункт меню. Попробуйте снова.")

        input("\nНажмите Enter, чтобы продолжить...")


if __name__ == "__main__":
    main()
