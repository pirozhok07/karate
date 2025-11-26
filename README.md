# karate
Отличный вопрос! Давайте создадим системный подход к тестированию приложения. Я расскажу, как правильно протестировать все модули и какие данные нужно создать.

1. Подготовка тестовых данных

Создадим тестовый скрипт

Файл: create_test_data.py в корне проекта

```python
import os
import django
from datetime import datetime, timedelta

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'karate_core.settings')
django.setup()

from django.contrib.auth.models import User
from tournaments.models import Tournament, WeightCategory
from participants.models import Participant, Registration
from judges.models import Judge, TournamentJudge
from brackets.models import Bracket, Match
from scoring.models import ScoringSession, JudgeAssignment, ScoreSheet

def create_test_users():
    """Создание тестовых пользователей"""
    users_data = [
        {'username': 'admin', 'email': 'admin@karate.ru', 'first_name': 'Администратор', 'last_name': 'Системы'},
        {'username': 'judge1', 'email': 'judge1@karate.ru', 'first_name': 'Иван', 'last_name': 'Петров'},
        {'username': 'judge2', 'email': 'judge2@karate.ru', 'first_name': 'Мария', 'last_name': 'Сидорова'},
        {'username': 'participant1', 'email': 'p1@karate.ru', 'first_name': 'Алексей', 'last_name': 'Иванов'},
        {'username': 'participant2', 'email': 'p2@karate.ru', 'first_name': 'Дмитрий', 'last_name': 'Смирнов'},
        {'username': 'participant3', 'email': 'p3@karate.ru', 'first_name': 'Сергей', 'last_name': 'Кузнецов'},
        {'username': 'participant4', 'email': 'p4@karate.ru', 'first_name': 'Анна', 'last_name': 'Попова'},
        {'username': 'participant5', 'email': 'p5@karate.ru', 'first_name': 'Елена', 'last_name': 'Васильева'},
        {'username': 'participant6', 'email': 'p6@karate.ru', 'first_name': 'Ольга', 'last_name': 'Новикова'},
        {'username': 'participant7', 'email': 'p7@karate.ru', 'first_name': 'Михаил', 'last_name': 'Федоров'},
        {'username': 'participant8', 'email': 'p8@karate.ru', 'first_name': 'Андрей', 'last_name': 'Морозов'},
    ]
    
    users = {}
    for user_data in users_data:
        user, created = User.objects.get_or_create(
            username=user_data['username'],
            defaults=user_data
        )
        if created:
            user.set_password('password123')
            user.save()
        users[user_data['username']] = user
    
    return users

def create_tournaments():
    """Создание тестовых турниров"""
    tournaments = []
    
    # Турнир по кумите
    kumite_tournament = Tournament.objects.create(
        name='Чемпионат Москвы по кумите 2024',
        description='Ежегодный чемпионат Москвы по каратэ-до (кумите)',
        tournament_type='kumite',
        start_date=datetime.now() + timedelta(days=7),
        end_date=datetime.now() + timedelta(days=8),
        location='Спорткомплекс "Олимпийский", Москва',
        status='upcoming',
        max_participants=100
    )
    tournaments.append(kumite_tournament)
    
    # Турнир по ката
    kata_tournament = Tournament.objects.create(
        name='Кубок России по ката 2024',
        description='Открытый кубок России по каратэ-до (ката)',
        tournament_type='kata',
        start_date=datetime.now() + timedelta(days=14),
        end_date=datetime.now() + timedelta(days=15),
        location='Дворец спорта "Лужники", Москва',
        status='upcoming',
        max_participants=50
    )
    tournaments.append(kata_tournament)
    
    return tournaments

def create_weight_categories(tournament):
    """Создание весовых категорий для турнира"""
    if tournament.tournament_type == 'kumite':
        categories = [
            {'name': 'До 60 кг', 'min_weight': 0, 'max_weight': 60, 'gender': 'M'},
            {'name': 'До 67 кг', 'min_weight': 60, 'max_weight': 67, 'gender': 'M'},
            {'name': 'До 75 кг', 'min_weight': 67, 'max_weight': 75, 'gender': 'M'},
            {'name': 'Свыше 75 кг', 'min_weight': 75, 'max_weight': 150, 'gender': 'M'},
            {'name': 'До 55 кг', 'min_weight': 0, 'max_weight': 55, 'gender': 'F'},
            {'name': 'Свыше 55 кг', 'min_weight': 55, 'max_weight': 150, 'gender': 'F'},
        ]
    else:
        categories = [
            {'name': 'Мужчины', 'min_weight': 0, 'max_weight': 150, 'gender': 'M'},
            {'name': 'Женщины', 'min_weight': 0, 'max_weight': 150, 'gender': 'F'},
        ]
    
    weight_categories = []
    for category_data in categories:
        category = WeightCategory.objects.create(
            tournament=tournament,
            **category_data
        )
        weight_categories.append(category)
    
    return weight_categories

def create_participants(users):
    """Создание участников"""
    participants_data = [
        {'user': users['participant1'], 'date_of_birth': '1995-03-15', 'gender': 'M', 'weight': 65.5, 'belt_level': 'brown', 'club': 'Секция каратэ "Самурай"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC001'},
        {'user': users['participant2'], 'date_of_birth': '1998-07-22', 'gender': 'M', 'weight': 70.2, 'belt_level': 'black', 'club': 'Клуб "Будокан"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC002'},
        {'user': users['participant3'], 'date_of_birth': '1993-11-05', 'gender': 'M', 'weight': 78.8, 'belt_level': 'brown', 'club': 'Спортшкола "Триумф"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC003'},
        {'user': users['participant4'], 'date_of_birth': '1997-01-30', 'gender': 'F', 'weight': 52.3, 'belt_level': 'black', 'club': 'Клуб "Сакура"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC004'},
        {'user': users['participant5'], 'date_of_birth': '1996-09-14', 'gender': 'F', 'weight': 58.7, 'belt_level': 'brown', 'club': 'Секция каратэ "Самурай"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC005'},
        {'user': users['participant6'], 'date_of_birth': '1994-12-08', 'gender': 'M', 'weight': 62.1, 'belt_level': 'black', 'club': 'Клуб "Будокан"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC006'},
        {'user': users['participant7'], 'date_of_birth': '1999-04-18', 'gender': 'M', 'weight': 74.5, 'belt_level': 'brown', 'club': 'Спортшкола "Триумф"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC007'},
        {'user': users['participant8'], 'date_of_birth': '1995-08-25', 'gender': 'F', 'weight': 53.9, 'belt_level': 'black', 'club': 'Клуб "Сакура"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC008'},
    ]
    
    participants = []
    for data in participants_data:
        participant = Participant.objects.create(**data)
        participants.append(participant)
    
    return participants

def create_judges(users):
    """Создание судей"""
    judges_data = [
        {'user': users['judge1'], 'judge_category': 'national', 'judge_level': 'a', 'license_number': 'JUDGE001', 'license_expiry': '2025-12-31', 'country': 'Россия', 'association': 'Федерация каратэ России', 'experience_years': 8},
        {'user': users['judge2'], 'judge_category': 'international', 'judge_level': 'referee', 'license_number': 'JUDGE002', 'license_expiry': '2025-12-31', 'country': 'Россия', 'association': 'Федерация каратэ России', 'experience_years': 12},
    ]
    
    judges = []
    for data in judges_data:
        judge = Judge.objects.create(**data)
        judges.append(judge)
    
    return judges

def create_registrations(participants, tournament, weight_categories):
    """Регистрация участников на турнир"""
    registrations = []
    
    # Распределяем участников по весовым категориям
    for i, participant in enumerate(participants):
        # Выбираем подходящую весовую категорию
        suitable_categories = [
            cat for cat in weight_categories 
            if cat.gender == participant.gender 
            and cat.min_weight <= participant.weight <= cat.max_weight
        ]
        
        if suitable_categories:
            category = suitable_categories[0]
            registration = Registration.objects.create(
                participant=participant,
                tournament=tournament,
                weight_category=category,
                status='approved'
            )
            registrations.append(registration)
    
    return registrations

def create_brackets(tournament, weight_categories):
    """Создание турнирных сеток"""
    brackets = []
    
    for category in weight_categories[:2]:  # Создаем сетки для первых двух категорий
        bracket = Bracket.objects.create(
            tournament=tournament,
            weight_category=category,
            name=f'Сетка {category.name}',
            bracket_type='single_elimination',
            status='draft'
        )
        brackets.append(bracket)
    
    return brackets

def main():
    """Основная функция создания тестовых данных"""
    print("Создание тестовых данных...")
    
    # Создаем пользователей
    users = create_test_users()
    print("✓ Пользователи созданы")
    
    # Создаем турниры
    tournaments = create_tournaments()
    print("✓ Турниры созданы")
    
    # Для каждого турнира создаем полный набор данных
    for tournament in tournaments:
        print(f"\n--- Настройка турнира: {tournament.name} ---")
        
        # Весовые категории
        weight_categories = create_weight_categories(tournament)
        print(f"✓ Весовые категории созданы: {len(weight_categories)}")
        
        # Участники
        participants = create_participants(users)
        print(f"✓ Участники созданы: {len(participants)}")
        
        # Судьи
        judges = create_judges(users)
        print(f"✓ Судьи созданы: {len(judges)}")
        
        # Регистрации
        registrations = create_registrations(participants, tournament, weight_categories)
        print(f"✓ Регистрации созданы: {len(registrations)}")
        
        # Турнирные сетки
        brackets = create_brackets(tournament, weight_categories)
        print(f"✓ Турнирные сетки созданы: {len(brackets)}")
        
        # Назначаем судей на турнир
        for judge in judges:
            TournamentJudge.objects.create(
                tournament=tournament,
                judge=judge,
                role='tatami',
                is_approved=True
            )
        print("✓ Судьи назначены на турнир")
    
    print("\n✅ Все тестовые данные успешно созданы!")
    print("\nДанные для входа:")
    print("Администратор: admin / password123")
    print("Судья 1: judge1 / password123")
    print("Участник 1: participant1 / password123")

if __name__ == '__main__':
    main()
```

2. Запуск создания тестовых данных

```bash
python create_test_data.py
```

3. План тестирования всех модулей

🔹 1. Тестирование модуля турниров

Что проверять:

· Создание турниров разных типов (кумите/ката)
· Настройка весовых категорий
· Статусы турниров (предстоящий, в процессе, завершен)

Количество для теста:

· 2 турнира (1 кумите + 1 ката)
· 4-6 весовых категорий на турнир

🔹 2. Тестирование модуля участников

Что проверять:

· Регистрация участников с разными характеристиками
· Разные уровни поясов, веса, клубы
· Статусы регистраций (ожидание, подтверждена, отклонена)

Количество для теста:

· 8-10 участников
· По 4-5 регистраций на турнир

🔹 3. Тестирование модуля судей

Что проверять:

· Судьи разных категорий и уровней
· Назначение судей на турниры
· Разные роли на турнире (главный, угловой и т.д.)

Количество для теста:

· 2-3 судьи
· По 2 назначения на турнир

🔹 4. Тестирование модуля сеток

Что проверять:

· Автоматическая генерация сеток
· Разные типы сеток (олимпийская система)
· Статусы сеток (черновик, опубликована, в процессе)

Количество для теста:

· 2 сетки (по одной на весовую категорию)
· Автоматическая генерация матчей

🔹 5. Тестирование модуля оценок

Что проверять:

· Создание сессий оценивания
· Назначение судей на сессии
· Заполнение оценочных листов
· Расчет результатов

Количество для теста:

· 1-2 сессии оценивания
· По 2 назначения судей на сессию

4. Пошаговое руководство по тестированию

📋 Шаг 1: Проверка админки

```bash
python manage.py runserver
```

Перейдите на http://localhost:8000/admin и проверьте:

· Все ли модели отображаются
· Можно ли создавать/редактировать записи
· Работает ли поиск и фильтрация

📋 Шаг 2: Проверка API endpoints

Турниры:

```bash
# Получить список турниров
curl http://localhost:8000/api/tournaments/tournaments/

# Получить конкретный турнир
curl http://localhost:8000/api/tournaments/tournaments/1/
```

Участники:

```bash
# Список участников
curl http://localhost:8000/api/participants/participants/

# Регистрации
curl http://localhost:8000/api/participants/registrations/
```

📋 Шаг 3: Тестирование workflow

1. Создайте турнир через админку
2. Зарегистрируйте участников на турнир
3. Создайте сетку и сгенерируйте матчи
4. Назначьте судей на сессию оценивания
5. Заполните оценки и определите победителя

📋 Шаг 4: Проверка связей

Убедитесь, что:

· Участники правильно привязаны к турнирам через регистрации
· Матчи правильно привязаны к сеткам
· Оценки правильно привязаны к матчам и судьям

5. Команды для проверки данных

```bash
# Проверить количество записей в базе
python manage.py shell -c "
from tournaments.models import Tournament
from participants.models import Participant, Registration
from judges.models import Judge
from brackets.models import Bracket, Match
from scoring.models import ScoringSession

print(f'Турниры: {Tournament.objects.count()}')
print(f'Участники: {Participant.objects.count()}')
print(f'Регистрации: {Registration.objects.count()}')
print(f'Судьи: {Judge.objects.count()}')
print(f'Сетки: {Bracket.objects.count()}')
print(f'Матчи: {Match.objects.count()}')
print(f'Сессии оценивания: {ScoringSession.objects.count()}')
"
```

6. Рекомендуемый объем тестовых данных

Модуль Минимально Рекомендуемо Для полного теста
Турниры 1 2 3-4
Участники 4 8 16-32
Судьи 1 2 4-6
Сетки 1 2 4-6
Матчи 2 4-8 16-32
Сессии оценивания 1 2 4-8

Такой объем данных позволит полноценно протестировать все функции приложения без избыточности.

Ошибка UNIQUE constraint failed: participants_participant.user_id возникает потому, что мы пытаемся создать несколько участников с одним и тем же пользователем. В модели Participant поле user - это OneToOneField, что означает, что один пользователь может быть только одним участником.

Давайте исправим скрипт создания тестовых данных:

Исправленный скрипт create_test_data.py

```python
import os
import django
from datetime import datetime, timedelta

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'karate_core.settings')
django.setup()

from django.contrib.auth.models import User
from tournaments.models import Tournament, WeightCategory
from participants.models import Participant, Registration
from judges.models import Judge, TournamentJudge
from brackets.models import Bracket, Match
from scoring.models import ScoringSession, JudgeAssignment, ScoreSheet

def clear_test_data():
    """Очистка старых тестовых данных"""
    print("Очистка старых тестовых данных...")
    ScoringSession.objects.all().delete()
    JudgeAssignment.objects.all().delete()
    ScoreSheet.objects.all().delete()
    TournamentJudge.objects.all().delete()
    Registration.objects.all().delete()
    Match.objects.all().delete()
    Bracket.objects.all().delete()
    Participant.objects.all().delete()
    Judge.objects.all().delete()
    WeightCategory.objects.all().delete()
    Tournament.objects.all().delete()
    
    # Удаляем только тестовых пользователей (кроме админа)
    User.objects.exclude(username='admin').delete()
    print("✓ Старые данные очищены")

def create_test_users():
    """Создание тестовых пользователей"""
    users_data = [
        {'username': 'admin', 'email': 'admin@karate.ru', 'first_name': 'Администратор', 'last_name': 'Системы'},
        {'username': 'judge1', 'email': 'judge1@karate.ru', 'first_name': 'Иван', 'last_name': 'Петров'},
        {'username': 'judge2', 'email': 'judge2@karate.ru', 'first_name': 'Мария', 'last_name': 'Сидорова'},
        {'username': 'participant1', 'email': 'p1@karate.ru', 'first_name': 'Алексей', 'last_name': 'Иванов'},
        {'username': 'participant2', 'email': 'p2@karate.ru', 'first_name': 'Дмитрий', 'last_name': 'Смирнов'},
        {'username': 'participant3', 'email': 'p3@karate.ru', 'first_name': 'Сергей', 'last_name': 'Кузнецов'},
        {'username': 'participant4', 'email': 'p4@karate.ru', 'first_name': 'Анна', 'last_name': 'Попова'},
        {'username': 'participant5', 'email': 'p5@karate.ru', 'first_name': 'Елена', 'last_name': 'Васильева'},
        {'username': 'participant6', 'email': 'p6@karate.ru', 'first_name': 'Ольга', 'last_name': 'Новикова'},
        {'username': 'participant7', 'email': 'p7@karate.ru', 'first_name': 'Михаил', 'last_name': 'Федоров'},
        {'username': 'participant8', 'email': 'p8@karate.ru', 'first_name': 'Андрей', 'last_name': 'Морозов'},
    ]
    
    users = {}
    for user_data in users_data:
        user, created = User.objects.get_or_create(
            username=user_data['username'],
            defaults=user_data
        )
        if created:
            user.set_password('password123')
            user.save()
            print(f"✓ Создан пользователь: {user_data['username']}")
        else:
            print(f"✓ Пользователь уже существует: {user_data['username']}")
        users[user_data['username']] = user
    
    return users

def create_participants(users):
    """Создание участников с проверкой существования"""
    participants_data = [
        {'user': users['participant1'], 'date_of_birth': '1995-03-15', 'gender': 'M', 'weight': 65.5, 'belt_level': 'brown', 'club': 'Секция каратэ "Самурай"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC001'},
        {'user': users['participant2'], 'date_of_birth': '1998-07-22', 'gender': 'M', 'weight': 70.2, 'belt_level': 'black', 'club': 'Клуб "Будокан"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC002'},
        {'user': users['participant3'], 'date_of_birth': '1993-11-05', 'gender': 'M', 'weight': 78.8, 'belt_level': 'brown', 'club': 'Спортшкола "Триумф"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC003'},
        {'user': users['participant4'], 'date_of_birth': '1997-01-30', 'gender': 'F', 'weight': 52.3, 'belt_level': 'black', 'club': 'Клуб "Сакура"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC004'},
        {'user': users['participant5'], 'date_of_birth': '1996-09-14', 'gender': 'F', 'weight': 58.7, 'belt_level': 'brown', 'club': 'Секция каратэ "Самурай"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC005'},
        {'user': users['participant6'], 'date_of_birth': '1994-12-08', 'gender': 'M', 'weight': 62.1, 'belt_level': 'black', 'club': 'Клуб "Будокан"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC006'},
        {'user': users['participant7'], 'date_of_birth': '1999-04-18', 'gender': 'M', 'weight': 74.5, 'belt_level': 'brown', 'club': 'Спортшкола "Триумф"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC007'},
        {'user': users['participant8'], 'date_of_birth': '1995-08-25', 'gender': 'F', 'weight': 53.9, 'belt_level': 'black', 'club': 'Клуб "Сакура"', 'country': 'Россия', 'city': 'Москва', 'license_number': 'LIC008'},
    ]
    
    participants = []
    for data in participants_data:
        # Проверяем, существует ли уже участник для этого пользователя
        participant, created = Participant.objects.get_or_create(
            user=data['user'],
            defaults=data
        )
        if created:
            participants.append(participant)
            print(f"✓ Создан участник: {data['user'].get_full_name()}")
        else:
            # Обновляем данные существующего участника
            for key, value in data.items():
                if key != 'user':  # Не обновляем пользователя
                    setattr(participant, key, value)
            participant.save()
            participants.append(participant)
            print(f"✓ Обновлен участник: {data['user'].get_full_name()}")
    
    return participants

def create_judges(users):
    """Создание судей с проверкой существования"""
    judges_data = [
        {'user': users['judge1'], 'judge_category': 'national', 'judge_level': 'a', 'license_number': 'JUDGE001', 'license_expiry': '2025-12-31', 'country': 'Россия', 'association': 'Федерация каратэ России', 'experience_years': 8},
        {'user': users['judge2'], 'judge_category': 'international', 'judge_level': 'referee', 'license_number': 'JUDGE002', 'license_expiry': '2025-12-31', 'country': 'Россия', 'association': 'Федерация каратэ России', 'experience_years': 12},
    ]
    
    judges = []
    for data in judges_data:
        judge, created = Judge.objects.get_or_create(
            user=data['user'],
            defaults=data
        )
        if created:
            judges.append(judge)
            print(f"✓ Создан судья: {data['user'].get_full_name()}")
        else:
            judges.append(judge)
            print(f"✓ Судья уже существует: {data['user'].get_full_name()}")
    
    return judges

def create_tournaments():
    """Создание тестовых турниров"""
    tournaments = []
    
    # Турнир по кумите
    kumite_tournament, created = Tournament.objects.get_or_create(
        name='Чемпионат Москвы по кумите 2024',
        defaults={
            'description': 'Ежегодный чемпионат Москвы по каратэ-до (кумите)',
            'tournament_type': 'kumite',
            'start_date': datetime.now() + timedelta(days=7),
            'end_date': datetime.now() + timedelta(days=8),
            'location': 'Спорткомплекс "Олимпийский", Москва',
            'status': 'upcoming',
            'max_participants': 100
        }
    )
    tournaments.append(kumite_tournament)
    if created:
        print("✓ Создан турнир по кумите")
    else:
        print("✓ Турнир по кумите уже существует")
    
    # Турнир по ката
    kata_tournament, created = Tournament.objects.get_or_create(
        name='Кубок России по ката 2024',
        defaults={
            'description': 'Открытый кубок России по каратэ-до (ката)',
            'tournament_type': 'kata',
            'start_date': datetime.now() + timedelta(days=14),
            'end_date': datetime.now() + timedelta(days=15),
            'location': 'Дворец спорта "Лужники", Москва',
            'status': 'upcoming',
            'max_participants': 50
        }
    )
    tournaments.append(kata_tournament)
    if created:
        print("✓ Создан турнир по ката")
    else:
        print("✓ Турнир по ката уже существует")
    
    return tournaments

def create_weight_categories(tournament):
    """Создание весовых категорий для турнира"""
    if tournament.tournament_type == 'kumite':
        categories = [
            {'name': 'До 60 кг', 'min_weight': 0, 'max_weight': 60, 'gender': 'M'},
            {'name': 'До 67 кг', 'min_weight': 60, 'max_weight': 67, 'gender': 'M'},
            {'name': 'До 75 кг', 'min_weight': 67, 'max_weight': 75, 'gender': 'M'},
            {'name': 'Свыше 75 кг', 'min_weight': 75, 'max_weight': 150, 'gender': 'M'},
            {'name': 'До 55 кг', 'min_weight': 0, 'max_weight': 55, 'gender': 'F'},
            {'name': 'Свыше 55 кг', 'min_weight': 55, 'max_weight': 150, 'gender': 'F'},
        ]
    else:
        categories = [
            {'name': 'Мужчины', 'min_weight': 0, 'max_weight': 150, 'gender': 'M'},
            {'name': 'Женщины', 'min_weight': 0, 'max_weight': 150, 'gender': 'F'},
        ]
    
    weight_categories = []
    for category_data in categories:
        category, created = WeightCategory.objects.get_or_create(
            tournament=tournament,
            name=category_data['name'],
            defaults=category_data
        )
        weight_categories.append(category)
        if created:
            print(f"✓ Создана весовая категория: {category_data['name']}")
    
    return weight_categories

def create_registrations(participants, tournament, weight_categories):
    """Регистрация участников на турнир"""
    registrations = []
    
    for participant in participants:
        # Выбираем подходящую весовую категорию
        suitable_categories = [
            cat for cat in weight_categories 
            if cat.gender == participant.gender 
            and cat.min_weight <= participant.weight <= cat.max_weight
        ]
        
        if suitable_categories:
            category = suitable_categories[0]
            registration, created = Registration.objects.get_or_create(
                participant=participant,
                tournament=tournament,
                weight_category=category,
                defaults={'status': 'approved'}
            )
            if created:
                registrations.append(registration)
                print(f"✓ Зарегистрирован участник: {participant.user.get_full_name()} в категории {category.name}")
    
    return registrations

def create_brackets(tournament, weight_categories):
    """Создание турнирных сеток"""
    brackets = []
    
    for category in weight_categories[:2]:  # Создаем сетки для первых двух категорий
        bracket, created = Bracket.objects.get_or_create(
            tournament=tournament,
            weight_category=category,
            defaults={
                'name': f'Сетка {category.name}',
                'bracket_type': 'single_elimination',
                'status': 'draft'
            }
        )
        if created:
            brackets.append(bracket)
            print(f"✓ Создана сетка: {category.name}")
    
    return brackets

def assign_judges_to_tournament(judges, tournament):
    """Назначение судей на турнир"""
    for judge in judges:
        assignment, created = TournamentJudge.objects.get_or_create(
            tournament=tournament,
            judge=judge,
            defaults={
                'role': 'tatami',
                'is_approved': True
            }
        )
        if created:
            print(f"✓ Назначен судья: {judge.user.get_full_name()} на турнир")

def main():
    """Основная функция создания тестовых данных"""
    print("=" * 50)
    print("СОЗДАНИЕ ТЕСТОВЫХ ДАННЫХ")
    print("=" * 50)
    
    # Очищаем старые тестовые данные (опционально)
    # clear_test_data()
    
    # Создаем пользователей
    print("\n1. СОЗДАНИЕ ПОЛЬЗОВАТЕЛЕЙ")
    users = create_test_users()
    
    # Создаем участников
    print("\n2. СОЗДАНИЕ УЧАСТНИКОВ")
    participants = create_participants(users)
    
    # Создаем судей
    print("\n3. СОЗДАНИЕ СУДЕЙ")
    judges = create_judges(users)
    
    # Создаем турниры
    print("\n4. СОЗДАНИЕ ТУРНИРОВ")
    tournaments = create_tournaments()
    
    # Для каждого турнира создаем полный набор данных
    for i, tournament in enumerate(tournaments, 1):
        print(f"\n5.{i}. НАСТРОЙКА ТУРНИРА: {tournament.name}")
        
        # Весовые категории
        weight_categories = create_weight_categories(tournament)
        
        # Регистрации
        registrations = create_registrations(participants, tournament, weight_categories)
        
        # Турнирные сетки
        brackets = create_brackets(tournament, weight_categories)
        
        # Назначаем судей на турнир
        assign_judges_to_tournament(judges, tournament)
    
    print("\n" + "=" * 50)
    print("✅ ВСЕ ТЕСТОВЫЕ ДАННЫЕ УСПЕШНО СОЗДАНЫ!")
    print("=" * 50)
    
    print("\n🔐 ДАННЫЕ ДЛЯ ВХОДА:")
    print("Администратор: admin / password123")
    print("Судья 1: judge1 / password123") 
    print("Участник 1: participant1 / password123")
    
    print("\n🌐 ДЛЯ ТЕСТИРОВАНИЯ:")
    print("1. Админка: http://localhost:8000/admin")
    print("2. API турниров: http://localhost:8000/api/tournaments/tournaments/")
    print("3. API участников: http://localhost:8000/api/participants/participants/")

if __name__ == '__main__':
    main()
```

Ключевые исправления:

1. Использовал get_or_create() вместо create() для всех моделей
2. Добавил проверки существования записей перед созданием
3. Убрал очистку данных по умолчанию (раскомментируйте clear_test_data() если нужно полное обновление)
4. Добавил информативные сообщения о процессе создания

Запуск исправленного скрипта:

```bash
python create_test_data.py
```

Если нужно полностью очистить и пересоздать данные:

Раскомментируйте строку в функции main():

```python
# Очищаем старые тестовые данные
clear_test_data()
```

Проверка создания данных:

После запуска скрипта проверьте через админку:

```bash
python manage.py runserver
```

Перейдите на http://localhost:8000/admin и убедитесь, что:

· ✅ Созданы 2 турнира (кумите и ката)
· ✅ Созданы 8 участников
· ✅ Созданы 2 судьи
· ✅ Созданы весовые категории
· ✅ Участники зарегистрированы на турниры
· ✅ Созданы турнирные сетки

Теперь скрипт можно запускать многократно без ошибок уникальности!
