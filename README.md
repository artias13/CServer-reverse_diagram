# CServer-reverse_diagram

## CServer Diagram

<p align="center">
    <img src="images/b4_reverse.drawio.png" alt="dfd1_diagram.png"/>
</p>

## Примерное описание функций:

В данном MD описаны основные функции test_large.bin (CServer):

### Function 1: sub_401760 (SecureNetworkDataTransmissionLoop):
Функция предназначена для облегчения зашифрованной передачи сообщений по сети, расшифровки полученных сообщений, выполнения удаленных команд, полученных с сервера, и предоставления соответствующей информации о состоянии системы.
- Инициирование Windows Socket API (WSA) вызовом WSAStartup(0x202u, &winsockData);
- Создание буфера шифрования и загрузка процедур шифрования с помощью вызова функции multiRC4EncryptAndLoadProcedures;
- Подготовка буфера данных путем сбора и форматирования системной информации, а затем отправка данных через защищенный сокет;
- Если отправленные данные валидны и соответствуют ответу «status_success», функция создает новый поток и выполняет полученные от сервера удаленные команды в этом потоке с помощью вызова функции RemoteCommandExecution;
- Если отправленные данные недействительны или ответ не соответствует «status_success», функция сделает паузу на определенный период (15 или 10 секунд в зависимости от этапа) перед повторной попыткой передачи данных.
- Функция работает в while цикле, который продолжается до тех пор, пока функция не будет принудительно завершена или сама программа не будет закрыта. 

### Function 2: sub_402510() (RC4DecryptAndLoadProcedures):
Эта функция использует функции (`LoadLibraryA` и `GetProcAddress`). Она предназначена для обфускации вызовов процедурам из DLL , необходимых для работы троянца. Вызывается в функции № 1 SecureNetworkCommunicationLoop.
- Начинает с расшифровки буфера с помощью алгоритма шифрования RC4 несколько раз с разными ключами и длинами. RC4 (Rivest Cipher 4) является симметричным потоковым шифром - он использует один и тот же ключ для шифрования и дешифрования. Буфер после расшифровки предположительно содержит имя динамической библиотеки и имена некоторых процедур.
- После расшифровки загружается динамическая библиотека с помощью функции `LoadLibraryA` и присваивает хэндл библиотечного модуля части буфера шифра(`cipherBuffer + 36`). Затем он получает адреса нескольких процедур в библиотеке с помощью функции `GetProcAddress` и присваивает эти адреса различным частям буфера шифра.
- Обеспечивает загрузку и доступ к процедурам из динамической библиотеки в зашифрованном виде. Имена библиотек и процедур зашифрованы, предположительно, с целью их обфускации.

## Function 2.1: sub_4028A0() (RC4_Cipher_Processing):
Функция предполагает выполнение криптографических операций. Основное назначение - шифрование или дешифрование данных с помощью алгоритма шифрования RC4.
- Принимает три аргумента - cipherBuffer, encodedString и input_length. Цель функции - ускорить процесс RC4 за счет предварительной буферизации элементов и использования минимального количества циклов при побайтовой операции шифрования. Функция qmemcpy копирует содержимое буфера шифра в массив состояний. Затем, в цикле, управляемом оператором 'do while', каждый байт в закодированной строке XORed (exclusive OR-ed) с псевдослучайным байтом из массива состояний, и результат записывается обратно в закодированную строку.
- Изменяет элементы массива состояний и выполняет XOR каждого байта закодированной строки с элементами массива состояний. На выходе получается подсчет обработанных байтов.

### Function 3: sub_4027A0() (processInputNumberAndSetString):
Функция должна принимать на вход число, а затем выполнять некоторую строковую операцию на основе значения входного числа. Точное назначение не совсем ясно.
- В качестве аргументов функция принимает целое число 'inputNumber' и символьный указатель 'outputString'.
- Если inputNumber равен 1, она копирует строку 'aSv' в 'outputString' с помощью функции 'strcpy' и устанавливает returnValue в 0.
- Если inputNumber не 1, то из inputNumber вычитается 2 и присваивается returnValue. Далее, если inputNumber равен 2, он получает длину 'aPxE77y', прибавляет к ней 1 и присваивает ее returnValue. Затем с помощью функции 'qmemcpy' копируем строку 'aPxME77y' в 'outputString'.
- Возвращает значение returnValue.

### Function 4: sub_402EE0() (GatherAndFormatSystemInformation):
Эта функция используется для сбора информации о системе, вызывается внутри while цикла в SecureNetworkCommunicationLoop.
- Сбор различной информации о системе
- Получает имя хоста
- Получает IP-адрес
- Получает информацию о версии Windows
- Получает имя компьютера
- Объединяет всю собранную информацию в один буфер

### Function 4.1: sub_403220() (GetAccountNameAndDomainViaSid):
Функция ищет имя учетной записи и домен получателя, используя идентификатор безопасности (SID) либо токена доступа текущего потока, либо токена доступа текущего процесса, если у текущего потока нет токена доступа. Затем эта информация копируется в параметры 'recipientDomain' и 'accountName'. 
- Инициализирует некоторые локальные переменные и очищает буферы.
- Получает хэндл доступа к токену доступа текущего потока с доступом TOKEN_QUERY (0x8). Если произошла ошибка, проверяется, является ли ошибка ERROR_NO_TOKEN (1008). Если это так, то вместо этого извлекается хэндл доступа к маркеру доступа текущего процесса.
- Получает информацию о маркере доступа с помощью функции GetTokenInformation(). В частности, она получает информацию о TokenUser, которая представляет учетную запись пользователя, связанную с маркером доступа.
- Использует SID, содержащийся в полученной информации о токене, для поиска имени учетной записи и домена, с которым связан этот SID, с помощью функции LookupAccountSidA().
- Если поиск имени учетной записи успешен, имя учетной записи копируется в параметр 'accountName', а соответствующее имя домена - в параметр 'recipientDomain'. 

### Function 4.2: sub_403360() (QueryAndAppendRegistryValueAndStatus):
Функция делает запрос определенных значений из реестра Windows, а именно 'SubKey', 'ValueName' и 'A8h' из ветки HKEY_CURRENT_USER.
- Открываем ключ реестра в ветке HKEY_CURRENT_USER в соответствии с 'SubKey'
- запрашивает размер заданного значения 'ValueName' в открытом ключе реестра
- Выделить место в памяти в соответствии с размером запрашиваемого значения
- Обнулить выделенное пространство памяти
- Запросить значение 'ValueName' из реестра и сохранить его в выделенном пространстве памяти.
- Запросить значение 'A8h' дважды, сначала запросив его размер, а затем сохранив его в буфере состояния (StatusBuffer)
-  запрос прошел успешно, добавьте запрошенное значение в выходной буфер.
- Если значение statusBuffer возвращается равным 1, присваиваем строку «On» тексту statusText; если 0, присваиваем строку «Off» тексту statusText
- Добавьте строку statusText в выходной буфер.
- Деаллокация выделенной памяти

### Function 4.3: sub_4031F0() (isX64orIPFArchitecture):
Функции получает информацию о системе и проверить, является ли архитектура процессора 9 или 6. Функция `FillSystemInfo` заполняет структуру `_SYSTEM_INFO` информацией о текущей системе, включая сведения о процессоре. Член `wProcessorArchitecture` этой структуры указывает архитектуру процессора установленного оборудования. Согласно официальной документации Microsoft, 9 соответствует x64 (AMD или Intel), а 6 - Intel Itanium Processor Family (IPF).
- объявляет структуру `_SYSTEM_INFO` под названием `v1`
- заполняет ее системной информацией с помощью функции `FillSystemInfo`
- проверяет, равен ли член `wProcessorArchitecture` из `v1` 9 или 6.
- если архитектура процессора равна 9 или 6, функция вернет TRUE
- в противном случае она вернет FALSE.

### Function 4.4: sub_4031B0() (FillSystemInfo):
Функция заполняет структуру SYSTEM_INFO правильной системной информацией в соответствии с окружением, в котором она работает. В 64-битной среде она пытается достичь более высокой точности, пытаясь использовать функцию `GetNativeSystemInfo`.
- проверяет, не является ли указатель на структуру SYSTEM_INFO NULL
- Если указатель не NULL, она переходит к следующим операциям. 
- использует функцию GetModuleHandleW для получения хэндла к указанному модулю, где модуль, вероятно, является библиотекой динамической компоновки (DLL)
- используется функция GetProcAddress для получения адреса функции 'GetNativeSystemInfo', которая находится в ранее загруженном модуле. 
- Функция `GetNativeSystemInfo` получает информацию о текущей системе для приложения, работающего под WOW64 (Windows on Windows 64). 
- Если система работает под управлением процесса WOW64, функция предоставляет более точную информацию о системе, например, о том, работает ли процессор как 32-битный или 64-битный.
- Если функция `GetNativeSystemInfo` не найдена в модуле, он возвращается к использованию функции `GetSystemInfo`, которая просто извлекает информацию о системе, независимо от запущенного процесса.


### Function 5: sub_402950() (prepare_and_write_message_to_buffer):
Функция форматирует строку и беззнаковое целое число в буфер, предоставленный вызывающей функцией. Строка и целое число, предположительно, являются глобальными переменными, учитывая, что они имеют префикс с подчеркиванием, который является обычным соглашением об именовании глобальных переменных в C. Поэтому ожидается, что функция запишет в буфер строку asc_4073E8 и целое число dword_407624.
- В качестве параметра она принимает указатель на буфер. Предположительно, этот буфер выделяется и поддерживается вне функции, и он предоставляется функции для того, чтобы она могла записать в него некоторые данные.
- Функция использует функцию «sprintf» для записи в буфер глобальной строки «asc_4073E8» и глобального беззнакового целого числа «dword_407624». 
- Функция «sprintf» используется для форматирования строки и целого числа в одну строку, которая записывается в буфер. Формат вывода задается строкой формата «%s%u», которая указывает, что сначала должна быть записана строка, а затем целое число без знака.
- Функция возвращает количество символов, записанных в буфер.

### Function 6: sub_401160() (base32_encode):
Функция кодирует входной буфер (_BYTE inputBuffer1) в выходной буфер с кодировкой Base32 (_BYTE inputBuffer2), используя предоставленную длину (formatted_system_info_ptr). Функция побайтно обрабатывает входной буфер, кодируя двоичные данные в Base32 и сохраняя их в выходном буфере. 
- Функция начинает с проверки того, что входные буферы не являются нулевыми. 
- Затем она считывает байты из входного буфера 'inputBuffer2'. 
- Для обработки данных различной длины используется переключатель, а для извлечения соответствующих символов в кодировке base32 - побитовые операции. 
- Специальные случаи для длин от 1 до 5 обрабатываются отдельно, с добавлением '32', когда это необходимо. 
- Цикл продолжается до тех пор, пока не будет обработан весь входной буфер. 
- Наконец, функция добавляет нулевой символ в конец выходного буфера, чтобы отметить конец строки в кодировке Base32.

### Function 7: sub_402A50() (format_and_append_strings):
Функция форматирует строку (Buffer), вставив в нее целое число (a2) и несколько строковых констант, хранящихся в известных местах памяти. Кроме того, она добавляет в буфер строку (a3). Она возвращает 0 независимо от успеха операции.
- В качестве первых двух аргументов она принимает символьный указатель (Buffer) и целое число (a2), а в качестве третьего аргумента - постоянный символьный указатель (a3).
- Затем форматирует Buffer, вставляя целое число 'a2' в строковую константу, хранящуюся в ячейке памяти 0x407200.
- Затем заполняет выход (Buffer), добавляя различные известные строки, хранящиеся в определенных местах памяти (unk_407210, unk_40722C, unk_40723C, unk_4072B4, unk_4072D8, unk_407314).
- Затем в конец этого форматированного буфера добавляется строка 'a3'.
- И наконец, в конец буфера добавляется строка, хранящаяся в ячейке памяти asc_407478.
- Все эти операции приводят к значению результата 0, которое в итоге возвращается.

### Function 8: sub_4034D0() (SendDataOverSecureSocket):
Функция занимается обработкой жизненного цикла сокетного соединения: создание, подключение, передача данных и закрытие. Судя по операциям функции, она создает сокет, устанавливает параметры сокета (например, таймаут соединения), соединяется с сервером на порту 443 (обычно используется для HTTPS-соединений), отправляет некоторые начальные данные, возможно, выполняет согласование SSL/TLS из-за возможной обработки ошибки 12045 (ERROR_INTERNET_INVALID_CA, часто связанной с SSL/TLS-соединениями), отправляет дополнительные данные и, наконец, закрывает соединение.
- создает сокет и сохраняет его хэндл.
- устанавливает параметры сокета, в данном случае таймаут соединения
- соединяет этот сокет с сервером на порту 443
- Отправляет некоторые начальные данные по этому соединению
- обрабатывает случай ошибки, когда SSL/TLS CA недействителен. Проходит через ряд операций, которые могут быть связаны с переговорами по SSL/TLS.
- Продолжает отправлять данные до тех пор, пока не будет отправлено указанное количество данных или не произойдет ошибка
- Закрывает соединения.
- Возвращает 1, если кажется, что все операции успешно завершены; в противном случае закрывает соответствующие хэндлы и возвращает 0

### Function 9: sub_401960() (RemoteCommandExecution // ExecuteSocketDataOperations is the same)
Функция имеет бесконечный цикл, который продолжает выполнять действия в зависимости от полученного параметра. Внутри цикла она подготавливает данные, отправляет их через защищенный сокет, а затем определяет свои действия на основе полученного ответа. Эти действия включают в себя:
- сбор и отправку информации о процессе обратно в систему управления
- завершение процессов
- создание потоков, выполняющих определенные задачи (включая выполнение заданной команды)
- запись в файл. 
В коде также включены функции временной задержки, чтобы избежать обнаружения и вести себя как доброкачественный процесс.
Каждый случай внутри оператора switch, похоже, соответствует определенному типу команды, которую может обработать данная функция. Функция 'RC4_Cipher_Processing', по-видимому, отвечает за расшифровку полученных данных (возможно, команды), которые затем обрабатываются.

## Function 9.1: sub_4024B0() (pause_for_random_time):
Функция вводит задержку в выполнение программы. Период этой задержки является частично случайным, а частично основан на входном аргументе. В частности, задержка равна сумме входного аргумента и модуля случайного числа и 6000 миллисекунд (6 секунд). Таким образом, точная продолжительность задержки будет меняться от вызова к вызову, что вносит определенную долю непредсказуемости в программу.
- Сначала объявляется целочисленная переменная v1.
- Затем присваивает v1 случайное число, сгенерированное с помощью функции rand().
- Наконец, она вызывает функцию Sleep, стандартную функцию языка C, которая приостанавливает работу программы на заданное количество миллисекунд. Количество миллисекунд ожидания равно сумме входного аргумента функции и результата операции «v1 % 6000». 

### Function 9.2: sub_401F20() (CollectAndSendProcessInformation)
Функция предназначена для сбора полного снимка всех запущенных процессов и некоторых их свойств, таких как идентификатор процесса, идентификатор родительского процесса, имя исполняемого файла. Она также потенциально может получить пути к файлам, связанным с модулями процессов, сохраняя все эти данные в буфере.
- Функция начинается с создания снимка процесса с помощью `CreateProcessSnapshot` и инициализации нескольких переменных.
- Затем она извлекает первый процесс из моментального снимка с помощью `RetrieveFirstProcessInformation`.
- В цикле do-while он собирает всю необходимую информацию, такую как идентификатор процесса, идентификатор родительского процесса и имя исполняемого файла.
- Затем эта информация сохраняется в буфере под названием processInfoBuffer.
- Функция открывает каждый процесс, перечисляет его модули и получает путь к файлу модуля. Если перечислить модули не удается, она получает имя файла выполнения процесса.
- После этого она переходит к следующему процессу, используя `GetNextProcessFromSnapshot`, и продолжает это до тех пор, пока не останется ни одного процесса.
- Завершает свой снимок процесса отправкой данных с помощью`secureDataTransmission`.
- Наконец, он закрывает хэндл снапшота с помощью `CloseHandle`.

### Function 9.3: sub_4020E0() (TerminateProcessAndSendStatus)
Функция завершает процесс на основе заданного идентификатора процесса (targetProcessId), а затем безопасно отправляет статус завершения (используя secureDataTransmission). 
- Объявление переменной HANDLE для хранения хэндла целевого процесса, копии этого хэндла и переменной status (terminationStatus) для хранения статуса завершения.
- Функция OpenProcess, которая является API Windows и позволяет получить хэндл процесса с заданным идентификатором процесса. Возвращенный хэндл хранится в переменной 'processHandle'. При этом запрашиваются все возможные права доступа к процессу (0x1F0FFF), флаг 'inherit handle' установлен в FALSE (0), а 'идентификатор процесса' равен 'targetProcessId'.
- Копия хэндла процесса назначается на 'processHandleCopy'.
- Функция проверяет, действителен ли хэндл процесса, и если да, то вызывает функцию TerminateProcess (функция Windows API, завершающая указанный процесс). Если завершение процесса прошло успешно, TerminateStatus устанавливается в 1.
- Далее функция вызывает secureDataTransmission для безопасной передачи данных о статусе завершения.
- В конце функция закрывает хэндл процесса с помощью CloseHandle (функция Windows API, закрывающая хэндл открытого объекта) и возвращает статус CloseHandle: TRUE в случае успеха; FALSE в противном случае.

### Function 9.4: sub_402150() (CreateThreadForCmd):
Функция создает pipe и новый поток. Это указывает на то, что она может быть потенциально использована в многопоточном приложении или в модели сервер-клиент для межпроцессного взаимодействия. 
- Объявляется структура _SECURITY_ATTRIBUTES для определения атрибутов безопасности трубы. Поля структуры nLength, lpSecurityDescriptor и bInheritHandle имеют определенные значения. В соответствии с этими значениями новый объект трубы будет разрешать наследование рукоятки, но не будет предоставлять дескриптор безопасности.
- Затем выполняется вызов функции CreatePipe. Это функция Windows API, которая создает анонимную трубу и возвращает хэндлы для чтения и записи на концах трубы.
- После этого происходит вызов функции CreateThread. Эта функция также является функцией Windows API. Она используется для создания нового потока в вызывающем процессе. Здесь вновь созданный поток начнет выполнение, вызвав функцию ExecuteCmdWithMonitoring. STACK[0x21C], предположительно, будет содержать некоторые параметры для этой функции, хранящиеся в стеке программы.
- Затем главный поток засыпает на 0x1F4u (что в десятичном исчислении равно 500), вероятно, на микросекунды, с помощью функции Sleep. Это может быть сделано для того, чтобы дать вновь созданному потоку достаточно времени для запуска и выполнения начальных операций.

### Function 9.4.1: sub_4021B0() (ExecuteCmdWithMonitoring):
Функция может быть использована для отслеживания и анализа поведения различных процессов.
- Инициализирует атрибуты безопасности для создания новой pipe.
- Устанавливает необходимые параметры для создания нового процесса 'cmd.exe', включая перенаправление stdout (стандартный выход) и stderr (стандартная ошибка) нового процесса в созданную им трубу.
- Запускает новый процесс 'cmd.exe'.
- Считывает данные из трубы, которые по сути являются выходами stdout и stderr процесса 'cmd.exe'.
- Отправляет считанные данные в функцию 'secureDataTransmission' для безопасной передачи данных.
- Отключает и очищает все дескрипторы и трубы, когда считывается определенная строка 'str2'.

### Function 9.5: sub_4023A0() (WriteWideStringToFile):
Функция предназначена для записи строки пользователя/сообщения в файл или канал.
- Многобайтовый строковый массив `multiByte String` размером 509 инициализируется и обнуляется.
- Функция извлекает текущую используемую кодовую страницу OEM-производителя с помощью функции `GetOEMCP()`.
- Широкая символьная строка, на которую указывает "wide Character String", преобразуется в многобайтовую строку с использованием кодовой страницы OEM с помощью "WideCharToMultiByte()". Результат этого преобразования затем сохраняется в `многобайтовой строке`.
- Содержимое "многобайтовой строки" записывается в файл с помощью "WriteFile()". Дескриптор файла (местоположения), в который должна быть записана строка, удаляется из `hWritePipe.year`.


### Function 10: sub_4040F0() (secureDataTransmission):
Функция используется в сетевом контексте, в частности, для обеспечения безопасности передачи данных. Псевдокод, по-видимому, выполняет создание буфера данных для передачи по соединению, шифрование буфера и безопасную передачу этих данных. Это может быть использовано в сценариях, где необходима безопасная передача данных, например, SSL или HTTPS.
- Функция сначала инициализирует два локальных буфера (`actionBuffer` и `sslBuffer`) и несколько неиспользуемых переменных
- Затем она выделяет буфер данных переданного размера с дополнительным интервалом в два символа
- Первый символ этого буфера инициализируется значением, передаваемым функции, а остальная часть заполняется исходными данными.
- Инициализируем `буфер данных` с помощью "initParam" и "srcData"
- Затем `Буфер данных` шифруется с использованием "RC4_CipherAlgorithm`
- Затем к зашифрованному "буферу данных" добавляется номер с помощью функции "appendToBufferWithNumber"
- Далее он форматируется и добавляется в буфер с помощью функции "format_and_append_to_buffer"
- Наконец, используя "secureDataRequest", данные передаются безопасно с включенными "actionBuffer", "connectionParam", "sslBuffer", "DataBuffer".

### Function 10.1: sub_402D40() (format_and_append_to_buffer):
Функция создает определенную строку в "Буфере", используя заданное целое число "a2" и указатель на символ "a3", добавляя при этом определенные жестко запрограммированные строки. Последовательно, она:
- Записывает целое число `a2` в `Buffer` в определенном формате, управляемом `YF`.
- Добавляет в `Buffer` несколько жестко закодированных строк.
- Если длина `a3` не равна нулю (т.е. это не пустая строка), он добавляет строку, на которую указывает `a3`, к `Buffer`.
- Он завершает работу с "буфером", добавляя еще несколько жестко закодированных строк.

### Function 10.2: sub_4036C0() (secureDataRequest)
Функция установление защищенного сетевого соединения (вероятно, SSL/TLS, как указано параметром "sslParam" и дополнительно подтверждено номером порта 443), выполнение определенного действия ("actionParam", по-видимому, определяет конкретное действие), считывание данных, полученных с это соединение помещается в буфер, а затем завершается соединением.
- Он инициирует дескриптор соединения с определенными параметрами.
- Затем значение тайм-аута для соединения устанавливается равным 10000.
- Он пытается подключиться к сети, в частности, через порт 443 (обычно для HTTPS).
- Если соединение установлено успешно, он выполняет определенное действие с определенными параметрами.
- Если действие завершается успешно, он либо считывает данные из соединения в буфер, либо обрабатывает ошибку 12045 (возможно, это связано с ошибкой сертификата), изменяя определенные флаги, и повторяет попытку.
- Функция продолжает считывать данные из соединения в буфер до тех пор, пока ничего не останется.
- Если все процессы завершены успешно, он проверяет, равен ли ответ в буфере `success`.
- На всех уровнях, если возникает какая-либо ошибка, функция устраняет ее, закрывая дескрипторы и возвращая 0.
