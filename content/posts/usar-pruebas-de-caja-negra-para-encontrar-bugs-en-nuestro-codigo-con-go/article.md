---
title: "Usar pruebas de caja negra para encontrar bugs en nuestro código con Go"
date: 2021-11-29T02:15:54-06:00
draft: true # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: 'Usar valores aleatorios en pruebas unitarias con Golang'
categories:
  - programming
tags:
  - go
  - testing
---

Las pruebas de caja negra:

Como ejemplo vamos a tener una función que se encargará de enmascarar un número de tarjeta de crédito. De manera que si de entrada tenemos `1234567890123456` la salida sea `1234XXXXXXXX3456`.

```golang
func Mask(card string) string {
	log.Println(card, len(card))

	start := card[0:4]
	between := strings.Repeat("X", 8)
	end := card[12:]
	res := fmt.Sprintf("%s%s%s", start, between, end)

	return res
}

```

Hacemos una prueba unitaria

```golang
func TestMask(t *testing.T) {
	t.Run("success", func(t *testing.T) {
		res := Mask("1234567890123456")

		if len(res) != 16 {
			t.Log("len should be 16")
			t.Fail()
		}

		if res != "1234XXXXXXXX3456" {
			t.Log("wrong output")
			t.Fail()
		}
	})
}
```

Ejecutamos la prueba y pasa. Tenemos nuestro código al 100% y aparentemente todo está bien. ¿El error? Estamos probando con inputs que sabemos que tendrán un resultado exitoso.

Golang tiene un paquete llamado quick, el cual nos brinda herramientas para hacer pruebas de caja negra.

Por ejemplo la función `Check` 

```golang
func Check(f interface{}, config *Config) error
```

Lo que hace esta función `Check` es tomar una función que retorne un bool (`f`) y ejecutarla `n` veces con valores aleatorios. Estos valores aleatorios, el número total `n` y demás valores son configurables a través de la estructura `Config`.

Como se mencionó con anterioridad, la función `f` debe retornar un bool. Pero ¿Qué debe realizar esta función? Dentro de esta función `f` debemos ejecutar nuestra función `Mask` y comprobar que los valores que esperamos de nuestra prueba unitaria se cumplan.

```golang
f := func(card string) bool {
	res := Mask(card)

	expectedLen := len(res) == 16

	return expectedLen
}
```

Como la función `f` se ejecutará un número de `n` veces utilizando valores aleatorios como argumentos no podemos esperar que el resultado de `Mask` sea siempre `123456789123456` por lo que la prueba del resultado esperado se omite.

Para crear la estructura `Config` podemos crear una función de ayuda como la siguiente:

```golang
func getConfig(t *testing.T) quick.Config {
	t.Helper()

	src := rand.NewSource(time.Now().Unix())

	return quick.Config{
		MaxCount: 100,
		Rand:     rand.New(src),
	}
}
```

De esta manera podemos reutilizar la función en caso de ser necesario o crear más pruebas de este tipo.

```golang
t.Run("random cards", func(t *testing.T) {
	f := func(card string) bool {
		res := Mask(card)

		expectedLen := len(res) == 16

		return expectedLen
	}
	cfg := getConfig(t)

	if err := quick.Check(f, &cfg); err != nil {
		t.Error(err)
	}
})
```

Dentro de este bloque de pruebas definimos nuestra función `f`, conseguimos la configuración y ejecutamos `quick.Check` utilizando estos valores. Comprobamos que en caso de error fallemos la prueba.

Al ejecutar el conjunto de pruebas vemos que tenemos un error:

```shell
2021/11/28 15:55:47 1234567890123456 16
2021/11/28 15:55:47 򶐡񦂅󡟗񸎏ꌞ򢬳񫭌򭵳򺓶󷢸⇴󟀱󦲚򹽉񱥢򠼯񇕛𯂴􆥆񕃩򮙖􃚬򠦞􈂰񉗡򹳯򶠃󍭙򧓦􁸺򌋺򓧐񁬼񉛯󜠬񌺼񘊳㶱𓺬󎙻鐎􋀓󸋈𑿽 175
--- FAIL: TestMask (0.00s)
    --- FAIL: TestMask/random_cards (0.00s)
        mask_test.go:34: #1: failed on input "\U000b6421\U00066085\U000e17d7\U0007838fꌞ\U000a2b33\U0006bb4c\U000add73\U000ba4f6\U000f78b8⇴\U000df031\U000e6c9a\U000b9f49\U00071962\U000a0f2f\U0004755b\U0002f0b4\U00106946\U000550e9\U000ae656\U001036ac\U000a099e\U001080b0\U000495e1\U000b9cef\U000b6803\U000cdb59\U000a74e6\U00101e3a\U0008c2fa\U000939d0\uf16c\U00041b3c\U000496ef\U000dc82c\U0004cebc\U000582b3㶱\U00013eac\U000ce67b鐎\U0010b013\U000f82c8\U00011ffd"
FAIL
exit status 1
FAIL	tctest	0.002s
go exited with status code 1
```

El primer log corresponde a la prueba controlada con un input de `1234567890123456`, mientras que el segundo log corresponde a una prueba ejecutada por Quick con un valor aleatorio. 

Se puede observar que el input de Mask fue una cadena de texto de 175 caracteres, muchos de los cuales ni siquiera se pueden interpretar en el terminal.

Al ejecutar las pruebas una vez más tenemos un error diferente. 

```shell
2021/11/28 17:34:14 1234567890123456 16
2021/11/28 17:34:14 򣝦 4
--- FAIL: TestMask (0.00s)
    --- FAIL: TestMask/random_cards (0.00s)
panic: runtime error: slice bounds out of range [12:4] [recovered]
	panic: runtime error: slice bounds out of range [12:4]

goroutine 8 [running]:
testing.tRunner.func1.2({0x50e1a0, 0xc000018300})
	/home/wako/.gvm/gos/go1.17.3/src/testing/testing.go:1209 +0x24e
testing.tRunner.func1()
	/home/wako/.gvm/gos/go1.17.3/src/testing/testing.go:1212 +0x218
panic({0x50e1a0, 0xc000018300})
	/home/wako/.gvm/gos/go1.17.3/src/runtime/panic.go:1038 +0x215
tctest.Mask({0xc000016264, 0x4})
	/home/wako/Datos/Codigo/Go/src/gotestquickmask/mask.go:14 +0x1b9
tctest.TestMask.func2.1({0xc000016264, 0x1})
	/home/wako/Datos/Codigo/Go/src/gotestquickmask/mask_test.go:25 +0x1e
reflect.Value.call({0x4fcf80, 0x523498, 0x4fb220}, {0x51873b, 0x4}, {0xc00000c0a8, 0x1, 0x1})
	/home/wako/.gvm/gos/go1.17.3/src/reflect/value.go:543 +0x814
reflect.Value.Call({0x4fcf80, 0x523498, 0xc00004a690}, {0xc00000c0a8, 0x1, 0x1})
	/home/wako/.gvm/gos/go1.17.3/src/reflect/value.go:339 +0xc5
testing/quick.Check({0x4fcf80, 0x523498}, 0x0)
	/home/wako/.gvm/gos/go1.17.3/src/testing/quick/quick.go:290 +0x233
tctest.TestMask.func2(0xc0001209c0)
	/home/wako/Datos/Codigo/Go/src/gotestquickmask/mask_test.go:33 +0x52
testing.tRunner(0xc0001209c0, 0x5234a0)
	/home/wako/.gvm/gos/go1.17.3/src/testing/testing.go:1259 +0x102
created by testing.(*T).Run
	/home/wako/.gvm/gos/go1.17.3/src/testing/testing.go:1306 +0x35a
exit status 2
FAIL	tctest	0.004s
go exited with status code 1

Sun Nov 28 17:34:14 2021
----------------
```

Dentro de este error podemos observar que nos indica que ocurrió un panic.

```shell
panic: runtime error: slice bounds out of range [12:4]
```

Esto se debe a que a la nuestra función intenta acceder al índice 12 de una cadena de texto de 4 caracteres.

De esta manera podemos observar como el pasar como input valores aleatorios nos ayuda a encontrar bugs los cuales tal vez no se habían considerado. 

Para solucionarlo tenemos que modificar nuestra función y volver a ejecutar las pruebas para comprobar la validez de nuestro programa.

Para asegurarnos que el resultado sea una cadena de texto de 16 caracteres y el que la función no caiga en un panic al acceder a los índices, podemos asegurarnos de validar el input de la función.

```golang
func Mask(card string) (string, error) {
	log.Println(card, len(card))

	if len(card) != 16 {
		return "", errors.New("card length should be 16")
	}

	start := card[0:4]
	between := strings.Repeat("X", 8)
	end := card[12:]
	res := fmt.Sprintf("%s%s%s", start, between, end)

	return res, nil
}
```

En esta nueva implementación agregamos un returno a nuestra función, este segundo retorno nos indicará sobre algún error dentro de nuestra función.

```golang
if len(card) != 16 {
  return "", errors.New("card length should be 16")
}
```

En caso de que la longitud del input sea diferente de 16 caracteres devolvemos un error. Esto evitará el panic y resultados no deseados.

Comprobamos el error durante la primera prueba

```golang
t.Run("success", func(t *testing.T) {
	res, err := Mask("1234567890123456")

	if err != nil {
		t.Log("err shuold be nil")
		t.Fail()
	}

	if len(res) != 16 {
		t.Log("len should be 16")
		t.Fail()
	}

	if res != "1234XXXXXXXX3456" {
		t.Log("wrong output")
		t.Fail()
	}
})
```

De esta manera aseguramos que en el caso de mandar como input `1234567890123456` no ocurrirá un error.

Para la segunda prueba debemos agregar la validación del error y utilizarlo como parte de la lógica del return de la función `f`.

```golang
t.Run("random cards", func(t *testing.T) {
	f := func(card string) bool {
		res, err := Mask(card)

		if err != nil {
			return len(res) == 0
		}

		expectedLen := len(res) == 16

		return expectedLen
	}
	cfg := getConfig(t)

	if err := quick.Check(f, &cfg); err != nil {
		t.Error(err)
	}
})
```

De esta manera ejecutamos las pruebas y observamos que ambas pruebas pasan.

```shell
2021/11/28 17:55:36 񂲫򈐾򓜧𻺹嫁򂆆🚒𾮹娅󪐿􋇢򦹸򽮝򖷌󹬰񦌤􋛝񁶠󙡯⾗񅗜󮍿򩡢 89
2021/11/28 17:55:36 𥋚󸈋򪋦񉸩 19
2021/11/28 17:55:36 񛃎󧳿𶒡𿓗󌳙򈱊􀍬󢪢񬔩𪼖𷞛𦴒󞞚񡨧󚥪󑑢񃵞𷱄 72
2021/11/28 17:55:36 󪼐󽠄󀳑𞀷󼦏򷜰𪂺񁪬􃭿񡒎򩝽뱣򒰛񋆛󕂴񭃆򊓨𤛊𾗗񉑤󹐸񣨨󈄀񘬾񦚠򵮤󙤎𛻻򧪴򅻠𻅋򀠰񍿫񔶈򫄣񠖑󧟞𜡼򪴉򫣜򰩮񋏊񦁙𯵩 175
2021/11/28 17:55:36 󖕎􈾌񢘘򃇂𯎊񐲂㻊󎅰󾀢􈸲􋁜恒򏡎򭣪񃨓𛦦󟌻񂺢ᢇ󹾍Ⅻ񭉳󻀯𑖬􆀷𧶲𪱺򬳼𒨱򐓙󶬺󘂴󉁣򼅬񵗎󮿼񦀘򉢽 148
2021/11/28 17:55:36 ꎓ򐎾򴘁񚝌󘨌񡭒񝠦񦵄󙬷𲝢󛨒񋹧𶵔𐪺򝪲𯱐񆚵񓄕𦈠򡳝󞳁𧉜󚔞󯺍𩾱󶟏񂃣𭖸򸂗𭷶󘗲󨂎󠠋󛶞 135
2021/11/28 17:55:36 򝹵񝳡󀯻񮩵𫔗󯼶񚡸񷎓况󡜘𸗟󝔸񗬿񢘥񳜖򯗲򠾠򙵮򷉾񊥋񰄼𭞒ᯡ󒈁 95
2021/11/28 17:55:36 񥏲𿾾𪅿󲝀󟥒􉢫𭠘񟾕󽥸話񄆛󜲚󤱷򰋚𒢙򯉆󆸪㫖󑳸𑬩𺶽񘁉巢㲶 93
2021/11/28 17:55:36 򈦄񻸙񘥇񀧲󜊷󷒹󼲧 28
2021/11/28 17:55:36 򇐦򎑴𒼸񟱬򛠂򟖒𧔙𧰺򿲴 36
2021/11/28 17:55:36 󂟅󎘕𬟈󆗡𾨴񮿀򂁼򙬖򌪬󂯑򇜇򀿸񎸩򡁇򿑌󡤋񽤽򧦥𵻧󗜑򩰷𐳤 91
PASS
ok  	tctest	0.002s
```

En apariencia todo quedó perfecto, las pruebas pasan con valores predecibles y aleatorios. Pero hay un problema por resolver, nuestra función `Mask` debería procesar tarjetas de crédito y devolverlas en un formato donde aparecen los primeros 4 dititos, seguidos de 8 X y finalizando con los últimos 4 digitos.

Para esto podemos valernos de una expresión regular para asegurarnos que nuestros datos se entregan en el formato correcto.

```golang
var (
	validMask = regexp.MustCompile(`^\d{4}[X]{8}\d{4}$`)
)
```

Aplicamos la validación a nuestra primera prueba:

```golang
t.Run("success", func(t *testing.T) {
	res, err := Mask("1234567890123456")

	if err != nil {
		t.Log("err shuold be nil")
		t.Fail()
	}

	if len(res) != 16 {
		t.Log("len should be 16")
		t.Fail()
	}

	if res != "1234XXXXXXXX3456" {
		t.Log("wrong output")
		t.Fail()
	}

	if !validMask.MatchString(res) {
		t.Log("res does not pass the regular expression")
		t.Fail()
	}
})
```

Comprobando que el resultado esperado cumpla con la expresión regular, de caso contrario la prueba deberá fallar.

Dentro de la función `f` validamos la expresión regular de la siguiente manera:

```golang
f := func(card string) bool {
	res, err := Mask(card)

	if err != nil {
		return len(res) == 0
	}

	expectedLen := len(res) == 16
	passRegex := validMask.MatchString(res)

	return expectedLen && passRegex
}
```

Al ejecutar las pruebas vemos el siguiente error:

```shell
2021/11/28 18:25:05 1234567890123456 16
2021/11/28 18:25:05 𻳩򱕽񘥁񥗠󠆉󧭲󊒢󁩍񆾜󐰢󒈛峓󤴦󬗻䤥򟻾󛸡񮜒󩕚󡹆򨠋򠤨󽨓􊧤񜦆򔶚󼄦󙌉𒭝 114
2021/11/28 18:25:05 񏡽񞺋󘗗􊍮򝾮񳢃󠩊󙘗󕱞򌗉󆦿򜳇󓑹򹇖񳮸₍𾚩􂫫󹷵򹇳񫱭򉑤󰽜󦔬𴪭򃾡𧢻򁳲𲁮􊁐񅉮񣼉󞶆󦫉󛈧򲐴 143
2021/11/28 18:25:05 𢹊𭟆򷣁񚙯󝦰􊞕󝎧񝁇򥗂񝯢򒢮򙙺󄔉񫰫󉻜󳯙񾜜񠴯񈼰񉮗񿻣򲛩𸶊񍖎񴩰󰿢񆝒禧󦿩󁇱򣹮񏯘􈆗󆚕󂇏򳟴򫒧񂒒򏂅񄛛䔅 162
2021/11/28 18:25:05 󔌜󞱪𯉝򏪮󯰃􋾤뱯󣛤󇆡𿢘ꕗ󺡼򗫹𻗜񼏜񧂹񢸔󒸈񇶐򩂓󵑄񚱤򬫭𤲛踇𙉹񎒌󬫱񺽒𕊡󟑲󫄼󹟕񺎬񓎵񍥔򝻾󧗨󆪚󞶩򎰤󌫪 165
2021/11/28 18:25:05 񧆜󤁭𱛒󅩞莩𳆦񁫚񮾁󌲽򗪘󱳉󳒒𴸽𗭨򏺮򫎘󯿄𐓨򷬭𕅿􈀲𑀣꭭좉𔣴𬕰󫎧񬷱򇦗򾊘񊫶򻅏񇖂򃉆񞏤񇘅 141
2021/11/28 18:25:05 􈁢򪄹񪋈𭎯񪥐𤌢𘶾񰫔︁򷅴 39
2021/11/28 18:25:05 񄌊㗎􊵥󸕞񲪃󓼷񛐇󔠑񟪵𨯈񑼫񯚯񮎑򴲓򾸮󭶘񨉘󫩊𤘚𥒛󒀗𤇝񫃢񗦑𬺧򘗜𽾿𼘎𐵵񠀠񒠛򋨺򲴸󖨒􋰴򂐳򬨭񯗵񿡱􍅌񋍪𔈾𗝮񯸀󏮆򲊂𻮓 187
2021/11/28 18:25:05 򞽩򿇯𬓋􀗽 16
--- FAIL: TestMask (0.00s)
    --- FAIL: TestMask/random_cards (0.00s)
        mask_test.go:49: #8: failed on input "\U0009ef69\U000bf1ef𬓋\U001005fd"
FAIL
exit status 1
FAIL	tctest	0.002s
go exited with status code 1
```

Vemos que la función de valores aleatorios se ejecutó varias veces. Las primeras veces se enviaron cadenas de texto cuya longitud no es 16, por lo tanto caen dentro de la validación de la longitud del input. Sin embargo vemos que la prueba fallida manda un texto de 16 caracteres:


```shell
2021/11/28 18:25:05 򞽩򿇯𬓋􀗽 16
```

El cual hace que la prueba falle ya que el resultado no cumple con nuestra expresión regular.

Para solucionar esto debemos implementar la expresión regular dentro de nuestra función:

```golang
func Mask(card string) (string, error) {
	log.Println(card, len(card))

	if len(card) != 16 {
		return "", errors.New("card length should be 16")
	}

	start := card[0:4]
	between := strings.Repeat("X", 8)
	end := card[12:]
	res := fmt.Sprintf("%s%s%s", start, between, end)


	if !validMask.MatchString(res) {
		return "", errors.New("card no regex")
	}

	return res, nil
}
```

En caso de que la validación de nuestra expresión regular falle vamos a retornar una cadena de texto vacía y un error.

```golang
if !validMask.MatchString(res) {
	return "", errors.New("card no regex")
}
```

Volvemos a ejecutar las pruebas y vemos que en esta ocasión todas nuestras pruebas pasan:

```golang
2021/11/28 18:32:18 򢧊▿򠰾񻐔򭇿򑶎󋺟󼈱 31
2021/11/28 18:32:18 󔄴򾀈񛐹󦟴󈖀󮪞𔿪񍇜񮙁𧕚𬢢򄾝񪡼󗜰 56
2021/11/28 18:32:18 񒧹񹳆񢆊򣀥򝅞󇏬򶆴򀕗􊅇񝟄󁆬󑪣񰛋𞚭󃈜 60
2021/11/28 18:32:18 󿕏򂻛󉫩󔽂򂲰䰸󺂵򖜲𾀵򡫒󺕾𵇠񸭶򼮖뚴󕎇񲼖󢀕󯻝􄤳񢏢񨢣俷𚙛񪾝񿘝𡈨򃪂𼸊󤄸򝽗㥺󩁨𬊴򺡽򲥈 140
2021/11/28 18:32:18 𼥸򜶿䰑𽎟񧈶򢝤󽽔󈓚򼒓󲅬򋆠򞟡釼󺽳󋐺𻈚򖛤𔓶򭭘󴰓񴊇𸊮򯗪𬁏񣶗򳞈􎵏𒦥胦񤹮򄌪󅘾񡁐򠎪򺨢񤅛񖲡򪢂󰳢󃰋񻻻 161
2021/11/28 18:32:18  0
2021/11/28 18:32:18 򷬅󂦕򀝰𯸖񼓐򰒶􊭥񦸘򙱠󂑦𮩗񄑦󚶌𚋁ᄴ򞝕󠊟𡼆 71
2021/11/28 18:32:18 񬱛󊛻𝙦񶘢򹲐򚀕𽟷𐘃􂒚𽷻񊈦󟢯񌌅򦫨򾼺򿾏񻈆󟶼񒬶񢊮𰞰𖓞𑘞󆽕񾨈񞇺򙖎񏈝񖼺񭇷 120
2021/11/28 18:32:18 󢬪𨋰󵪥􋤚󵂮򯔸򠟜󘓉􄳪򠟗񼰗󴤢􌳖򒟸񙮯񥯮򺉊󾭻󩻮򥲔򚑙򆸥񅒾򭭱󔷽󐚡򁭁𐊒󰇕𚓱򉕻򜻜𶀿񓘪򰒟򷆐􀸼񾉕 155
2021/11/28 18:32:18 􈤈񞈫󜼜󛛠򉖎뷆󝽻򍽀󮿢戝🧑򽷓󽸃𢹓󇜫񷨮򱦐𢴡 70
2021/11/28 18:32:18 󦔸򗯝򺌕񢏠򸍌󐯚򶦬񱄠񺭗񭟐񾟽񪄷񕧋󻴀󃊈񎺑󤅉򈊬򠳸􃨪󾲎񪼢𽖭񭿫𨳌괏󜢧󁗬󿏏󻧟 119
2021/11/28 18:32:18 󲯠𝹫󙹘򣉰񽇿𳛽򏀴 28
2021/11/28 18:32:18 􁠒󙟄󪪿񁣻𵒹񬴋󥨺򯉭𗮴񯪳񾸮񀬭򗤒󸸘󜟫𠿈򰅣
PASS
ok      tctest  0.006s
```

De esta manera podemos valernos de herramientas de pruebas de caja negra para poder comprobar los caos de uso de nuestras funciones. Y así poder evitar posibles bugs que podemos omitir por seguir el `Happy Path` de nuestra aplicación.

Pueden consultar el código guente del ejemplo [en el siguiente enlace de Github](https://github.com/4k1k0/examplesGo/tree/main/quickTest).
