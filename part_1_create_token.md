Цей туторіал написаний власником Macbook.
Тому всі поради будуть стосуватись користувачів Macbook.

План туторіалу:
1. Розробка токена.
2. Тестування токена.
3. Створення контракту для проведення crowdsale.


Частина 1. Розробка власного токена

Трохи теорії.
Смартконтракти на Ethereum пишуться на Solidity. Це мова, яка спеціально створена для написання деценетралізованих програм - програми, які виконуються на блокчейні.

Для розробки на Solidity я використовую середовище Remix - https://remix.ethereum.org. Це найкраще, що зараз існує. 
Це середовище підтримує найновіші версії Solidity, debugger та має інтелектуальні порадники: підказки щодо помилок, використання застарілих функцій Solidity,  підкреслення тексту.

Є ще багато різних інструментів для розробки смартконтрактів - налаштування локального блокчейну на власному комп’ютері, розробка юніт тестів для контрактів, під’єднання до блокчейну за допомогою фреймворків і написання децентралізованих додатків з юзер інтерфейсом.
Але до цього ми повернемось пізніше.

Тут ми будемо створювати smart contract на стандарті ERC20. Він найпоширеніший і дозволяє вашому токену підтримуватись будь-якими гаманцями та біржами (якщо ви звісно хочете, щоб ваш токен продавався на біржах). Код, який використовується тут, взятий з найбільш безпечної бібліотеки смарт контрактів - Zeppelin. Це дозволить вам вчитись на найкращих прикладах.

1. Перший крок
В лівому куті сторінки ви можете знайти кнопку, яка створює новий файл. Натискаємо і називаємо файл так, як хочете щоб називався ваш токен.
Я називаю свій MalCoin.sol.

2. Другий крок
Створюємо контракт MalCoin.

`pragma solidity ^0.4.21; // Зазначаємо останню стабільну версію Solidity

contract MalCoin {
}
`
Слово contract тут - це те ж саме, що і class в інших мовах програмування.

3. Третій крок
Наповнюємо contract базовим функціоналом.

Нам необхідна змінна, яка буде зберігати баланс кожного власника нашого токена.

mapping(address => uint256) public balances;

address - тип даних, у який можна зберігати адресу. Це може бути адреса гаманця власника токенів або адреса іншого smart contract.
uint256 - тип даних, у який можна зберігати лише додатні числа. Взагалі в Solidity все працює на uint - (unsigned int в С++ і С). Тут не має floating point чисел. uint256  можна писати як uint, але для кращої коментованості коду, я пишу постійно uint256.
mapping - це тип даних, який працює за прикладом асоціативних масивів, тобто словників key => value.
public - модифікатор, який дозволяє контракту, який наслідує contract MalCoin, використовувати цю змінну.
Синтаксис address => uint256 означає, що за ключем address знаходиться баланс власника токена.


Створюємо змінну в якій буде зберігатись загальна кількість токенів.

uint256 public totalSupply_;


Створюємо функцію, яка буде віддавати баланс конкретного власника токенів. Синтаксис функції схожий до того, який використовуєтсья в JS.

function balanceOf(address owner) public view returns (uint256) {
	return balances[owner];
}

view - модифікатор функції. Він каже компілятору, що ця функція не змінює нічого в блокчейні, а просто хоче отримати значення з mapping. Це допомагає оптимізувати роботу компілятора і зробити обчислення менш затратними.


Створюємо функцію, за допомогою якої можна подивитись totalSupply_ нашого контракту.

function totalSupply() public view returns (uint256) {
	return totalSupply_;
}


Як бачимо, поки що функціонал не дозволяє нам перекидувати токени на іншим користувачам. Тому напишемо функцію transfer.
`
function transfer(address _to, uint256 _value) public returns (bool) {
	// тут ми не дозволяємо перекидувати токени на нульову адресу
	require(_to != address(0));
	// тут перевіряємо чи є у відправника відповідна кількість токенів
	require(_value <= balances[msg.sender]);
	
	// зменшуємо баланс відправника на відпвідну суму
	balances[msg.sender] = balances[msg.sender] - _value;
	// збільшуємо баланс отримувача на відповідну суму
	balances[_to] = balances[_to] + _value;
	// івент про передачу токенів
	emit Transfer(msg.sender, _to, _value);
	return true;
}
`
У цій функції багато нового. Розглянемо усе по порядку.
`require` - функція, яка вбудована в Solidity і дозволяє припиняти виконання коду і внесення змін до блокчейну, якщо не справджуються умови зазначені в ній. Якщо (_to != address(0)) == true, то код виконуватиметься далі. Якщо буде fasle, то спрацює механізм викидання з функції - throw.
`emit Transfer(…)` - це event, який ми ще не написали. Івенти використовуються для того, щоб сповіщати про певні операції. Є деякі програми, які вміють їх виловлювати і показувати повідомлення для юзера. Наприклад - електронні гаманці вміють ловити івенти і показувати їх юзеру.
`msg.sender` - юзер, який визиває функцію. msg - це об’єкт, який вбудований у Solidity. працює як глобальна змінна.


Реалізуємо `event Transfer`.

`event Transfer(address indexed _from, address indexed _to, uint256 _value);`

В івентах є тільки сигнатура без реалізації.


Тепер наш контракт з базовим функціоналом виглядає таким чином.
`
contract MalCoin {
	mapping(address => uint256) balances;
	uint256 totalSupply_;
	
	event Transfer(address indexed _from, address indexed _to, uint256 _value);
	
	function totalSupply() public view returns (uint256) {
		return totalSupply_;
  	}

	function transfer(address _to, uint256 _value) public returns (bool) {
		require(_to != address(0));
		require(_value <= balances[msg.sender]);

		balances[msg.sender] = balances[msg.sender] - _value;
		balances[_to] = balances[_to] + _value;
		emit Transfer(msg.sender, _to, _value);
		return true;
	}

	function balanceOf(address _owner) public view returns (uint256) {
		return balances[_owner];
	}
}
`


4. Четвертий крок
Наповнюємо контракт повним функціоналом, який відповідатиме стандарту ERC20.

Є ситуації коли необхідно дозволити іншим користувачам чи контрактам користуватись вашим балансом токенів. Для цього ми створюємо mapping, який зберігатиме інформацію про те, скільки токенів один користувач дозволив використовувати іншому користувачу.

`mapping (address => mapping (address => uint256)) internal allowed;`

Цей mapping виглядає дивно. Але за допомогою нього можна робити наступне:

`
address public user = 0xdd870fa1b7c4700f2bd7f44238821c26f7392148
address public user1 = 0xca35b7d915458ef540ade6068dfe2f44e8fa733c
address public user2 = 0x14723a09acff6d2a60dcdf7aa4aff308fddc160c
address public someContract = 0x4b0897b0513fdc7c541b6d9d7e929c4e5364d2db

allowed[user][user1] = 1000;
allowed[user][user2] = 20000;
allowed[user][someContract] = 100000;
`

Як ви бачите, user дозволив user1 використовувати 1000 токенів, а user2 - 20000, a someContract - 100000.


Тепер напишемо функцію, яка буде встановлювати ліміт токенів, які будуть дозволені іншій особі для використання.

`
function approve(address _spender, uint256 _value) public returns (bool) {
	// Встановлюємо ліміт токенів
	allowed[msg.sender][_spender] = _value;
	// Івент про затвердження ліміту використання токенів
	emit Approval(msg.sender, _spender, _value);
	return true;
}
`

Реалізуємо `event Approval`.

`event Approval(address indexed owner, address indexed spender, uint256 value);`


За допомогою функції transfer ми можемо тільки перекидати токени з msg.sender до отримувача. Тепер за допомогою змінної allowed ми напишемо функцію, яка зможе перекидувати токени з будь-якого адреса на інший.

`
function transferFrom(address _from, address _to, uint256 _value) public returns (bool) {
	require(_to != address(0));
	require(_value <= balances[_from]);
	//  Тут ми перевіряємо чи _from надав дозвіл msg.sender на надсилання токенів 
	require(_value <= allowed[_from][msg.sender]);

	balances[_from] = balances[_from] - _value;
	balances[_to] = balances[_to] + _value;

	// Зменшуємо суму токенів дозволених для використання
	allowed[_from][msg.sender] = allowed[_from][msg.sender] - _value;
	emit Transfer(_from, _to, _value);
	return true;
}
`

Для того, щоб подивитись яку кількість токенів дозволено використовувати одному користувачу від імені іншого напишемо функцію allowance.

`
function allowance(address _owner, address _spender) public view returns (uint256) {
	return allowed[_owner][_spender];
}
`

Для того, щоб збільшити або зменшити Approval напишемо додаткові дві функції.

`
function increaseApproval(address _spender, uint _addedValue) public returns (bool) {
	allowed[msg.sender][_spender] = allowed[msg.sender][_spender] +_addedValue;
	emit Approval(msg.sender, _spender, allowed[msg.sender][_spender]);
	return true;
}

function decreaseApproval(address _spender, uint _subtractedValue) public returns (bool) {
	uint oldValue = allowed[msg.sender][_spender];
	if (_subtractedValue > oldValue) {
		allowed[msg.sender][_spender] = 0;
	} else {
		allowed[msg.sender][_spender] = oldValue - _subtractedValue;
	}
	emit Approval(msg.sender, _spender, allowed[msg.sender][_spender]);
	return true;
}
`


Після того як ми написали смарт контракт, зробимо його більш безпечним. Проблема будь-який числових змінних в тому, що для них виділяється конкретна кількість бітів. В Solidity можна створювати змінні, які можуть зберігати числа на 256 біт. Максимальне число, яке залізає в 256 біт - 115792089237316195423570985008687907853269984665640564039457584007913129639935.
Вам здається, що у вас ніколи такі числа не вийдуть, адже вони занадто великі? Але це число доволі просто отримати. Зараз покажу це на простому прикладі:
`
uint256 public num = 0;
uint256 public max = num - 1;
`
Все, у змінній max записано число, яке я зазначав вище. Перевірте це в remix.

Насправді є набагато більше проблем з числами, яких потрібно уникнути.
Для цього я використовую для будь-яких математичних операцій бібліотеку SafeMath. Вона робить операції надійними і захищає від переповнення значення змінних.

Разом з бібліотекою SafeMath наш код виглядатиме так:

`
pragma solidity ^0.4.21;

library SafeMath {

function mul(uint256 a, uint256 b) internal pure returns (uint256 c) {
	if (a == 0) {
		return 0;
	}
	c = a * b;
	assert(c / a == b);
	return c;
}

function div(uint256 a, uint256 b) internal pure returns (uint256) {
	// assert(b > 0); // Solidity automatically throws when dividing by 0
	// uint256 c = a / b;
	// assert(a == b * c + a % b); // There is no case in which this doesn't hold
	return a / b;
}

function sub(uint256 a, uint256 b) internal pure returns (uint256) {
	assert(b <= a);
	return a - b;
}

function add(uint256 a, uint256 b) internal pure returns (uint256 c) {
	c = a + b;
	assert(c >= a);
	return c;
}
}


contract MalCoin {
  using SafeMath for uint256;

  mapping(address => uint256) balances;

  uint256 totalSupply_;

mapping (address => mapping (address => uint256)) internal allowed;


  function totalSupply() public view returns (uint256) {
    return totalSupply_;
  }

  function transfer(address _to, uint256 _value) public returns (bool) {
    require(_to != address(0));
    require(_value <= balances[msg.sender]);

    balances[msg.sender] = balances[msg.sender].sub(_value);
    balances[_to] = balances[_to].add(_value);
    emit Transfer(msg.sender, _to, _value);
    return true;
  }

  function balanceOf(address _owner) public view returns (uint256) {
    return balances[_owner];
  }

  function transferFrom(address _from, address _to, uint256 _value) public returns (bool) {
    require(_to != address(0));
    require(_value <= balances[_from]);
    require(_value <= allowed[_from][msg.sender]);

    balances[_from] = balances[_from].sub(_value);
    balances[_to] = balances[_to].add(_value);
    allowed[_from][msg.sender] = allowed[_from][msg.sender].sub(_value);
    emit Transfer(_from, _to, _value);
    return true;
  }

  function approve(address _spender, uint256 _value) public returns (bool) {
    allowed[msg.sender][_spender] = _value;
    emit Approval(msg.sender, _spender, _value);
    return true;
  }

  function allowance(address _owner, address _spender) public view returns (uint256) {
    return allowed[_owner][_spender];
  }

  function increaseApproval(address _spender, uint _addedValue) public returns (bool) {
    allowed[msg.sender][_spender] = allowed[msg.sender][_spender].add(_addedValue);
    emit Approval(msg.sender, _spender, allowed[msg.sender][_spender]);
    return true;
  }

  function decreaseApproval(address _spender, uint _subtractedValue) public returns (bool) {
    uint oldValue = allowed[msg.sender][_spender];
    if (_subtractedValue > oldValue) {
      allowed[msg.sender][_spender] = 0;
    } else {
      allowed[msg.sender][_spender] = oldValue.sub(_subtractedValue);
    }
    emit Approval(msg.sender, _spender, allowed[msg.sender][_spender]);
    return true;
  }

}
`
