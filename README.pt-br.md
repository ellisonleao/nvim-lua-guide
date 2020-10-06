# Primeiros passos usando Lua no Neovim

## Introdução

A integração do Lua como uma linguagem de primeira classe dentro do Neovim está se configurando para ser uma de suas características matadoras. No entanto, a quantidade de material didático para aprender a escrever plug-ins em Lua não é tão grande quanto você encontraria para escrevê-los em Vimscript. Esta é uma tentativa de fornecer algumas informações básicas para ajudar e incentivar quem está começando agora.

Este guia assume que você está usando o [build noturno](https://github.com/neovim/neovim/releases/tag/nightly) mais recente do Neovim. Como a versão 0.5 do Neovim é uma versão em desenvolvimento, tenha em mente que algumas APIs nas quais estão sendo trabalhadas ativamente não são muito estáveis e podem mudar antes do lançamento.

### Aprendendo Lua

Se você ainda não está familiarizado com a linguagem, existem muitos recursos para começar:

- A página [Aprenda X em Y minutos sobre Lua](https://learnxinyminutes.com/docs/lua/) deve fornecer uma visão geral rápida dos fundamentos
- Se os vídeos são mais do seu agrado, Derek Banas tem um [tutorial de 1 hora sobre a linguagem](https://www.youtube.com/watch?v=iMacxZQMPXs)
- O [wiki de usuários do lua](http://lua-users.org/wiki/LuaDirectory) está cheio de informações úteis sobre todos os tipos de tópicos relacionados ao Lua
- O [manual oficial de referência para o Lua](https://www.lua.org/manual/5.1/) deve fornecer uma visão mais completo da linguagem

Deve-se notar também que o Lua é uma linguagem muito limpa e simples. É fácil de aprender, especialmente se você tiver experiência com linguagens de script semelhantes, como JavaScript. Você já deve conhecer mais o Lua do que imagina!

Nota: a versão do Lua que o Neovim embute é LuaJIT 2.1.0, que mantém a compatibilidade com Lua 5.1 (com algumas extensões 5.2)

### Tutoriais existentes para escrever em Lua com o Neovim

Alguns tutoriais já foram criados para ajudar as pessoas a escrever plug-ins em Lua. Alguns deles ajudaram bastante na redação deste guia. Muito obrigado aos seus autores.

- [teukka.tech - Do init.vim para init.lua](https://teukka.tech/luanvim.html)
- [2n.pl - Como escrever plug-ins neovim em Lua](https://www.2n.pl/blog/how-to-write-neovim-plugins-in-lua.md)
- [2n.pl - Como fazer interface do usuário para plug-ins neovim em Lua](https://www.2n.pl/blog/how-to-make-ui-for-neovim-plugins-in-lua)
- [ms-jpq - Tutorial Neovim Async](https://ms-jpq.github.io/neovim-async-tutorial/)

### Plugins complementares

- [Vimpeccable](https://github.com/svermeulen/vimpeccable) - Plugin para ajudar a escrever seu .vimrc em Lua
- [plenary.nvim](https://github.com/nvim-lua/plenary.nvim) - Todas as funções lua que não quero escrever duas vezes
- [popup.nvim](https://github.com/nvim-lua/popup.nvim) - Uma implementação da API Popup do vim no Neovim
- [nvim_utils](https://github.com/norcalli/nvim_utils)
- [nvim-luadev](https://github.com/bfredl/nvim-luadev) - console REPL / debug para plug-ins nvim lua
- [nvim-luapad](https://github.com/rafcamlet/nvim-luapad) - Bloco de rascunho neovim interativo em tempo real para mecanismo lua integrado
- [nlua.nvim](https://github.com/tjdevries/nlua.nvim) - Desenvolvimento Lua para Neovim
- [BetterLua.vim](https://github.com/euclidianAce/BetterLua.vim) - Melhor destaque da sintaxe Lua no Vim / NeoVim

## Onde colocar os arquivos Lua

Os arquivos Lua são normalmente encontrados dentro de uma pasta `lua/` em seu `runtimepath` (para a maioria dos usuários, isso significará `~/.config/nvim/lua` em sistemas *nix e `~/AppData/Local/nvim/lua` no Windows). Os globais `package.path` e `package.cpath` são ajustados automaticamente para incluir arquivos Lua nesta pasta. Isso significa que você pode `require()` esses arquivos como módulos Lua.

Vamos pegar a seguinte estrutura de pastas como exemplo:

```texto
📂 ~/.config/nvim
├── 📁 after
├── 📁 ftplugin
├── 📂 lua
│  ├── 🌑 myluamodule.lua
│  └── 📂 other_modules
│     ├── 🌑 anothermodule.lua
│     └── 🌑 init.lua
├── 📁 pack
├── 📁 plugin
├── 📁 syntax
└── 🇻 init.vim
```

O seguinte código Lua carregará `myluamodule.lua`:

```lua
require('myluamodule')
```

Observe a ausência de uma extensão `.lua`.

Da mesma forma, o carregamento de `other_modules/anothermodule.lua` é feito assim:

```lua
require('other_modules.anothermodule')
- ou
require('other_modules/anothermodule')
```

Os separadores de caminho são indicados por um ponto `.` ou por uma barra `/`.

Uma pasta contendo um arquivo `init.lua` pode ser solicitada diretamente, sem ter que especificar o nome do arquivo.

```lua
require('other_modules') -- loads other_modules/init.lua
```

Para mais informações: `:help lua-require`

#### Ressalvas

Ao contrário dos arquivos .vim, os arquivos .lua não são originados automaticamente de diretórios em seu `runtimepath`. Em vez disso, você deve obtê-los/solicitá-los do Vimscript. Existem planos para adicionar a opção de carregar um arquivo `init.lua` como uma alternativa para` init.vim`:

- [Problema nº 7895](https://github.com/neovim/neovim/issues/7895)
- [solicitação pull correspondente](https://github.com/neovim/neovim/pull/12235)

#### Dicas

Vários plug-ins Lua podem ter nomes de arquivo idênticos em sua pasta `lua/`. Isso pode levar a conflitos de espaço de nomes.

Se dois plug-ins diferentes têm um arquivo `lua/main.lua`, então fazer `require('main')` é ambíguo: qual arquivo fonte nós queremos?

Pode ser uma boa ideia atribuir um namespace à sua configuração ou ao seu plugin com uma pasta de nível superior, como: `lua/plugin_name/main.lua`

## Usando Lua a partir de Vimscript

### :lua

Esse comando executa um trecho do código Lua.

```vim
:lua require('myluamodule')
```

Scripts multilinhas são possíveis usando a sintaxe heredoc:

```vim
echo "Aqui está um pedaço maior do código Lua"

lua << EOF
local mod = require('mymodule')
local tbl = {1, 2, 3}

for k, v in ipairs(tbl) do
    mod.method(v)
end

print(tbl)
EOF
```


Veja também:

- `:help :lua`
- `:help :lua-heredoc`

#### Ressalvas

Você não obtém o realce de sintaxe correto ao escrever Lua em um arquivo .vim. Pode ser mais conveniente usar o comando `:lua` como um ponto de entrada para solicitar arquivos Lua externos.

### :luado

Este comando executa um pedaço de código Lua que atua em um intervalo de linhas no buffer atual. Se nenhum intervalo for especificado, todo o buffer será usado. Qualquer string que seja retornada (`return`) do pedaço de código é usada para determinar com o que cada linha deve ser substituída.

O seguinte comando substituiria todas as linhas no buffer atual com o texto `hello world`:

```vim
:luado return 'hello world'
```

Duas variáveis implícitas `line` e `linenr` também são fornecidas. `line` é o texto da linha sendo iterada, enquanto `linenr` é o seu número. O comando a seguir tornaria cada linha cujo número é divisível por 2 maiúsculas:

```vim
:luado if linenr % 2 == 0 then return line:upper() end
```

Veja também:

- `:help :luado`

###: luafile

Este comando fornece um arquivo Lua.

```vim
:luafile ~/foo/bar/baz/myluafile.lua
```

É análogo ao comando `:source` para arquivos .vim ou à função incorporada `dofile()` em Lua.

Veja também:

- `:help :luafile`

#### luafile vs require():

Você pode estar se perguntando qual é a diferença entre `lua require()` e `luafile` e se você deve usar um em vez do outro. Eles têm diferentes casos de uso:

- `require()`:
    - é uma função Lua embutida. Ele permite que você aproveite as vantagens do sistema de módulos de Lua
    - busca por módulos usando a variável `package.path` (como observado anteriormente, você pode `require()` scripts Lua localizados dentro da pasta `lua/` em seu `runtimepath`)
    - mantém registro de quais módulos foram carregados e evita que um script seja analisado e executado uma segunda vez. Se você alterar o arquivo que contém o código de um módulo e tentar `require()` uma segunda vez enquanto o Neovim está em execução, o módulo não será atualizado
- `:luafile`:
    - é um Ex command. Não suporta módulos
    - pega um caminho que é absoluto ou relativo ao diretório de trabalho da janela atual
    - executa o conteúdo de um script independentemente de ele ter sido executado antes

`:luafile` também pode ser útil se você deseja executar um arquivo Lua no qual está trabalhando:

```vim
:luafile %
```

### luaeval()

Essa função Vimscript embutida avalia uma string de expressão matemática Lua e retorna seu valor. Os tipos de dados Lua são convertidos automaticamente em tipos Vimscript (e vice-versa).

```vim
" Você pode armazenar o resultado em uma variável
let variable = luaeval('1 + 1')
echo variable
" 2
let concat = luaeval('"Lua".." eh ".."incrivel"')
echo concat
" 'Lua eh incrivel'

" Tabelas semelhantes a listas são convertidas para listas Vim
let list = luaeval('{1, 2, 3, 4}')
echo list[0]
" 1
echo list[1]
" 2
" Observe que, ao contrário das tabelas Lua, as listas do Vim são indexadas em 0

" Tabelas semelhantes a Dict são convertidas em dicionários Vim
let dict = luaeval('{foo = "bar", baz = "qux"}')
echo dict.foo
" 'bar'

" O mesmo vale para booleanas e nulidade
echo luaeval('true')
" v:true
echo luaeval('nil')
" v:null

" Você pode criar pseudônimos Vimscript para funções Lua
let LuaMathPow = luaeval('math.pow')
echo LuaMathPow(2, 2)
" 4
let LuaModuleFunction = luaeval('require("mymodule").myfunction')
call LuaModuleFunction()

" Também é possível passar funções Lua como valores para funções Vim
lua X = function(k, v) return string.format("%s:%s", k, v) end
echo map([1, 2, 3], luaeval("X"))
```

`luaeval()` recebe um segundo argumento opcional que permite que você passe dados para uma expressão matemática. Você pode então acessar esses dados de Lua usando o mágico `_A` global:

```vim
echo luaeval('_A[1] + _A[2]', [1, 1])
" 2

echo luaeval('string.format("Lua eh %s", _A)', 'incrivel')
" 'Lua eh incrivel'
```

Veja também:
- `:help luaeval()`

### v:lua

Essa variável global do Vim permite chamar funções Lua globais diretamente do Vimscript. Novamente, os tipos de dados do Vim são convertidos em tipos Lua e vice-versa.

```vim
call v:lua.print('Ola da Lua!')
" 'Ola da Lua!'

let scream = v:lua.string.rep('A', 10)
echo scream
" 'AAAAAAAAAA'

" Podemos requerir módulos
call v:lua.require('mymodule').myfunction()

" Que tal um bom statusline?
lua << EOF
function _G.statusline()
    local filepath = '%f'
    local align_section = '%='
    local percentage_through_file = '%p%%'
    return string.format(
        '%s%s%s',
        filepath,
        align_section,
        percentage_through_file
    )
end
EOF

set statusline=%!v:lua.statusline()

" Também funciona em mapeamentos de expressões
lua << EOF
function _G.check_back_space()
    local col = vim.fn.col('.') - 1
    if col == 0 or vim.fn.getline('.'):sub(col, col):match('%s') then
        return true
    else
        return false
    end
end
EOF

inoremap <silent> <expr> <Tab>
    \ pumvisible() ? '\<C-n>' :
    \ v:lua.check_back_space() ? '\<Tab>' :
    \ completion#trigger_completion()
```

Veja também:
- `:help v:lua`
- `:help v:lua-call`


#### Ressalvas

Esta variável só pode ser usada para chamar funções. O seguinte sempre gerará um erro:

`` `vim
" As funções de pseudônimo não funcionam
let LuaPrint = v:lua.print

" Acessar dicionários não funciona
echo v:lua.some_global_dict['key']

" Usar uma função como valor não funciona
echo map([1, 2, 3], v:lua.global_callback)
```

## O namespace do vim

Neovim expõe uma variável global `vim` que serve como um ponto de entrada para interagir com suas APIs de Lua. Ele fornece aos usuários uma "biblioteca padrão" estendida de funções, bem como vários submódulos.

Algumas funções e módulos notáveis incluem:

- `vim.inspect`: objetos Lua de impressão bonita (útil para inspecionar tabelas)
- `vim.regex`: use regexes do Vim através do Lua
- `vim.api`: módulo que expõe funções da API (a mesma API usada por plug-ins remotos)
- `vim.loop`: módulo que expõe a funcionalidade do loop de evento do Neovim (usando LibUV)
- `vim.lsp`: módulo que controla o cliente LSP integrado
- `vim.treesitter`: módulo que expõe a funcionalidade da biblioteca tree-sitter

Esta lista não é abrangente. Se você deseja saber mais sobre o que é disponibilizado pela variável `vim`, utilize `:help lua-stdlib` e `:help lua-vim`. Alternativamente, você pode fazer `:lua print(vim.inspect(vim))` para obter uma lista de cada módulo.

#### Dicas

Escrever `print(vim.inspect(x))` toda vez que você deseja inspecionar o conteúdo de um objeto pode ser muito tedioso. Pode valer a pena ter uma função de wrapper global em algum lugar da sua configuração:

```lua
function _G.dump(...)
    local objects = vim.tbl_map(vim.inspect, {...})
    print(unpack(objects))
end
```

Você pode então inspecionar o conteúdo de um objeto muito rapidamente em seu código ou na linha de comando:

```lua
dump({1, 2, 3})
```

```vim
:lua dump(vim.loop)
```


Além disso, você pode descobrir que às vezes faltam funções nativas do Lua em comparação com o que você encontraria em outras linguagens (por exemplo, `os.clock()` retorna apenas um valor em segundos, não milissegundos). Certifique-se de verificar o stdlib do Neovim (e `vim.fn`, falaremos mais sobre isso depois), ele provavelmente tem o que você está procurando.

## Usando Vimscript através do Lua

### vim.api.nvim_eval()

Esta função avalia uma string de expressão Vimscript e retorna seu valor. Os tipos de dados Vimscript são convertidos automaticamente em tipos Lua (e vice-versa).

É o equivalente em Lua da função `luaeval()` em Vimscript

```lua
-- Os tipos de dados são convertidos corretamente
print(vim.api.nvim_eval('1 + 1')) -- 2
print(vim.inspect(vim.api.nvim_eval('[1, 2, 3]'))) -- { 1, 2, 3 }
print(vim.inspect(vim.api.nvim_eval('{"foo": "bar", "baz": "qux"}'))) -- { baz = "qux", foo = "bar" }
print(vim.api.nvim_eval('v:true')) -- true
print(vim.api.nvim_eval('v:null')) -- nil
```

**A FAZER**: é possível que `vim.api.nvim_eval ()` retorne um `funcref`?

#### Ressalvas

Ao contrário de `luaeval()`, a utilização de `vim.api.nvim_eval()` não fornece uma variável `_A` implícita para passar dados para a expressão.

### vim.api.nvim_exec()

Esta função avalia um pedaço do código Vimscript. Leva em uma string contendo o código-fonte para ser executado e um booleano para determinar se a saída do código deve ser retornada pela função (você pode então armazenar a saída em uma variável, por exemplo).

```lua
local result = vim.api.nvim_exec(
[[
let mytext = 'hello world'

function! MyFunction(text)
    echo a:text
endfunction

call MyFunction(mytext)
]],
true)

print(result) -- 'hello world'
```

**A FAZER**: A documentação diz que o escopo do script (`s:`) é suportado, mas executar este trecho com uma variável com escopo do script gera um erro. Por quê?

### vim.api.nvim_command()

Esta função executa um comando ex. Ele recebe uma string contendo o comando a ser executado.

```lua
vim.api.nvim_command('new')
vim.api.nvim_command('wincmd H')
vim.api.nvim_command('set nonumber')
vim.api.nvim_command('%s/foo/bar/g')
```

Nota: `vim.cmd` é um pseudônimo mais curto para esta função

```lua
vim.cmd('buffers')
```

#### Dicas

Como você precisa passar strings para essas funções, muitas vezes acaba tendo que escapar das barras invertidas:

```lua
vim.cmd('%s/\\Vfoo/bar/g')
```

Strings literais são mais fáceis de usar, pois não requerem caracteres de escape:

```lua
vim.cmd([[%s/\Vfoo/bar/g]])
```

## Gerenciando opções do vim

### Usando funções API

Neovim fornece um conjunto de funções de API para definir (set) uma opção ou obter (get) seu valor atual:

- Opções globais:
    - `vim.api.nvim_set_option()`
    - `vim.api.nvim_get_option()`
- Opções locais de buffer:
    - `vim.api.nvim_buf_set_option()`
    - `vim.api.nvim_buf_get_option()`
- Opções locais da janela:
    - `vim.api.nvim_win_set_option()`
    - `vim.api.nvim_win_get_option()`

Eles recebem uma string contendo o nome da opção para definir/obter, bem como o valor que você deseja definir.

As opções booleanas (como `(no)number`) devem ser definidas como `true` (verdadeiro) ou `false` (falso):

```lua
vim.api.nvim_set_option('smarttab', false)
print(vim.api.nvim_get_option('smarttab')) -- false
```

Não deve ser surpresa que as opções de string devam ser definidas como uma string:

```lua
vim.api.nvim_set_option('selection', 'exclusive')
print(vim.api.nvim_get_option('selection')) -- 'exclusive'
```

As opções de número aceitam um número:

```lua
vim.api.nvim_set_option('updatetime', 3000)
print(vim.api.nvim_get_option('updatetime')) -- 3000
```

As opções de buffer local e janela local também precisam de um número de buffer ou de janela (usando `0` irá definir/obter a opção para o buffer/janela atual):

```lua
vim.api.nvim_win_set_option(0, 'number', true)
vim.api.nvim_buf_set_option(10, 'shiftwidth', 4)
print(vim.api.nvim_win_get_option(0, 'number')) -- true
print(vim.api.nvim_buf_get_option(10, 'shiftwidth')) -- 4
```

### Usando meta-acessores

Alguns meta-acessores estão disponíveis se você quiser definir opções de uma maneira mais "idiomática". Eles basicamente cobrem as funções de API acima e permitem que você manipule opções como se fossem variáveis:

- `vim.o.{option}`: opções globais
- `vim.bo.{opção}`: opções locais do buffer
- `vim.wo.{opção}`: opções locais da janela

```lua
vim.o.smarttab = false
print(vim.o.smarttab) -- false

vim.bo.shiftwidth = 4
print(vim.bo.shiftwidth) -- 4
```

Você pode especificar um número para opções locais de buffer e locais de janela. Se nenhum número for fornecido, o buffer/janela atual é usado:

```lua
vim.bo[4].expandtab = true -- same as vim.api.nvim_buf_set_option(4, 'expandtab', true)
vim.wo.number = true -- same as vim.api.nvim_win_set_option(0, 'number', true)
```

Veja também:
- `:help lua-vim-internal-options`

#### Ressalvas

**AVISO**: a seção a seguir é baseada em alguns experimentos que fiz. Os documentos não parecem mencionar esse comportamento e eu não verifiquei o código-fonte para verificar minhas afirmações.
**A FAZER**: Alguém pode confirmar isso?

Se você sempre lidou com opções usando o comando `:set`, o comportamento de algumas opções pode surpreendê-lo.

Essencialmente, as opções podem ser globais, locais para um buffer/janela ou ter um valor global **e** local.

O comando `:setglobal` define o valor global de uma opção.
O comando `:setlocal` define o valor local de uma opção.
O comando `:set` define o valor global **e** local de uma opção.

Aqui está uma tabela útil de `:help :setglobal`:


|                 Comando | valor global | valor local |
| ----------------------: | :----------: | :---------: |
|       :set option=value |     set      |     set     |
|  :setlocal option=value |      -       |     set     |
| :setglobal option=value |     set      |      -      |

Não há equivalente ao comando `:set` em Lua, você define uma opção global ou localmente.

Você pode esperar que a opção `number` seja global, mas a documentação a descreve como sendo "local para a janela". Essas opções são, na verdade, "fixas": seu valor é copiado da janela atual quando você abre uma nova.

Portanto, se você definisse a opção em `init.lua`, faria da seguinte forma:

```lua
vim.wo.number = true
```

Opções que são "locais para buffer" como `shiftwidth`, `expandtab` ou `undofile` são ainda mais confusas. Digamos que seu `init.lua` contém o seguinte código:

```lua
vim.bo.expandtab = true
```

Quando você inicia o Neovim e começa a editar, está tudo bem: pressionar `<Tab>` insere espaços em vez de um caractere de tabulação. Abra outro buffer e você de repente está de volta às guias...

Configurá-lo globalmente tem o problema oposto:

```lua
vim.o.expandtab = true
```

Desta vez, você insere guias ao iniciar o Neovim pela primeira vez. Abra outro buffer e o pressionar do `<Tab>` fará o que você espera.

Em suma, as opções que são "locais para buffer" devem ser definidas da seguinte maneira se você quiser o comportamento correto:

```lua
vim.bo.expandtab = true
vim.o.expandtab = true
```

Veja também:
- `:help :setglobal`
- `:help global-local`

**A FAZER**: Por que isso acontece? Todas as opções locais de buffer se comportam dessa maneira? Pode estar relacionado a [neovim/neovim#7658](https://github.com/neovim/neovim/issues/7658) e a [vim/vim#2390](https://github.com/vim/vim/issues/2390). Também para opções de janela local: [neovim/neovim#11525](https://github.com/neovim/neovim/issues/11525) e [vim/vim#4945](https://github.com/vim/vim/issues/4945)

## Gerenciando variáveis internas do vim

### Usando funções API

Assim como as opções, as variáveis internas têm seu próprio conjunto de funções de API:

- Variáveis globais (`g:`):
    - `vim.api.nvim_set_var()`
    - `vim.api.nvim_get_var()`
    - `vim.api.nvim_del_var()`
- Variáveis de buffer (`b:`):
    - `vim.api.nvim_buf_set_var()`
    - `vim.api.nvim_buf_get_var()`
    - `vim.api.nvim_buf_del_var()`
- Variáveis de janela (`w:`):
    - `vim.api.nvim_win_set_var()`
    - `vim.api.nvim_win_get_var()`
    - `vim.api.nvim_win_del_var()`
- Variáveis da página de tabulação (`t:`):
    - `vim.api.nvim_tabpage_set_var()`
    - `vim.api.nvim_tabpage_get_var()`
    - `vim.api.nvim_tabpage_del_var()`
- Variáveis Vim predefinidas (`v:`):
    - `vim.api.nvim_set_vvar()`
    - `vim.api.nvim_get_vvar()`

Com exceção das variáveis Vim predefinidas, elas também podem ser excluídas (o comando `:unlet` é o equivalente no Vimscript). Variáveis locais (`l:`), variáveis de script (`s:`) e argumentos de função (`a:`) não podem ser manipulados, pois só fazem sentido no contexto de um script Vim, Lua tem suas próprias regras de escopo.

Se você não está familiarizado com o que essas variáveis fazem, `:help internal-variables` as descreve em detalhes.

Essas funções recebem uma string contendo o nome da variável para definir/obter/excluir, bem como o valor para o qual você deseja defini-la.

```lua
vim.api.nvim_set_var('some_global_variable', { key1 = 'value', key2 = 300 })
print(vim.inspect(vim.api.nvim_get_var('some_global_variable'))) -- { key1 = "value", key2 = 300 }
vim.api.nvim_del_var('some_global_variable')
```

Variáveis que têm como escopo um buffer, uma janela ou uma página de tabulação também recebem um número (usando `0` irá definir/obter/excluir a variável para o buffer/janela/página de tabulação atual):

```lua
vim.api.nvim_win_set_var(0, 'some_window_variable', 2500)
vim.api.nvim_tab_set_var(3, 'some_tabpage_variable', 'hello world')
print(vim.api.nvim_win_get_var(0, 'some_window_variable')) -- 2500
print(vim.api.nvim_buf_get_var(3, 'some_tabpage_variable')) -- 'hello world'
vim.api.nvim_win_del_var(0, 'some_window_variable')
vim.api.nvim_buf_del_var(3, 'some_tabpage_variable')
```

### Usando meta-acessores

Variáveis internas podem ser manipuladas de forma mais intuitiva usando estes meta-acessores:

- `vim.g.{name}`: variáveis globais
- `vim.b.{name}`: variáveis de buffer
- `vim.w.{name}`: variáveis de janela
- `vim.t.{name}`: variáveis da página de tabulação
- `vim.v.{name}`: variáveis Vim predefinidas

```lua
vim.g.some_global_variable = {
    key1 = 'value',
    key2 = 300
}

print(vim.inspect(vim.g.some_global_variable)) -- { key1 = "value", key2 = 300 }
```

Para excluir uma dessas variáveis, simplesmente atribua `nil` a ela:

```lua
vim.g.some_global_variable = nil
```

#### Ressalvas

Ao contrário dos meta-acessores de opções, você não pode especificar um número para variáveis com escopo de buffer/janela/página de tabulação.

Além disso, você não pode adicionar/atualizar/excluir chaves de um dicionário armazenado em uma dessas variáveis. Por exemplo, este snippet de código Vimscript não funciona conforme o esperado:

```vim
let g:variable = {}
lua vim.g.variable.key = 'a'
echo g:variable
" {}
```

Este é um problema conhecido:

- [Issue #12544](https://github.com/neovim/neovim/issues/12544)

## Chamando funções Vimscript

### vim.call()

`vim.call()` chama uma função Vimscript. Pode ser uma função integrada do Vim ou uma função do usuário. Novamente, os tipos de dados são convertidos entre Lua e Vimscript.

Leva o nome da função seguido pelos argumentos que você deseja passar para essa função:

```lua
print(vim.call('printf', 'Hello from %s', 'Lua'))

local reversed_list = vim.call('reverse', { 'a', 'b', 'c' })
print(vim.inspect(reversed_list)) -- { "c", "b", "a" }

local function print_stdout(chan_id, data, name)
    print(data[1])
end

vim.call('jobstart', 'ls', { on_stdout = print_stdout })

vim.call('my#autoload#function')
```

Veja também:
- `:help vim.call()`

### vim.fn.{function}()

`vim.fn` faz exatamente a mesma coisa que `vim.call()`, mas se parece mais com uma chamada de função nativa do Lua:

```lua
print(vim.fn.printf('Hello from %s', 'Lua'))

local reversed_list = vim.fn.reverse({ 'a', 'b', 'c' })
print(vim.inspect(reversed_list)) -- { "c", "b", "a" }

local function print_stdout(chan_id, data, name)
    print(data[1])
end

vim.fn.jobstart('ls', { on_stdout = print_stdout })
```
Hashtags `#` não são caracteres válidos para identificadores em Lua, então funções de carregamento automático devem ser chamadas com esta sintaxe:

```lua
vim.fn['my#autoload#function']()
```

Veja também:
- `:help vim.fn`

#### Dicas

O Neovim possui uma extensa biblioteca de poderosas funções integradas que são muito úteis para plug-ins. Veja `:help vim-function` para uma lista alfabética e `:help function-list` para uma lista de funções agrupadas por tópico.

#### Ressalvas

Algumas funções do Vim que deveriam retornar uma booleana, retornam `1` ou` 0`. Isso não é um problema no Vimscript, já que `1` é verdadeiro e `0` falso, permitindo construções como estas:

```vim
if has('nvim')
    " do something...
endif
```

Em Lua, no entanto, apenas `false` e` nil` são considerados falsos, os números sempre avaliam como `true` independentemente de seu valor. Você deve verificar explicitamente se há `1` ou` 0`:

```lua
if vim.fn.has('nvim') == 1 then
    -- do something...
end
```

## Definindo mapeamentos

Neovim fornece uma lista de funções de API para definir, obter e excluir mapeamentos:

- Mapeamentos globais:
    - `vim.api.nvim_set_keymap()`
    - `vim.api.nvim_get_keymap()`
    - `vim.api.nvim_del_keymap()`
- Mapeamentos de buffer local:
    - `vim.api.nvim_buf_set_keymap()`
    - `vim.api.nvim_buf_get_keymap()`
    - `vim.api.nvim_buf_del_keymap()`

Vamos começar com `vim.api.nvim_set_keymap()` e `vim.api.nvim_buf_set_keymap()`

O primeiro argumento passado para a função é uma string contendo o nome do modo para o qual o mapeamento terá efeito:


| Valor da string        | Página de ajuda | Modos afetados                           | Equivalente no Vimscript  |
| ---------------------- | -------------   | ---------------------------------------- | --------------------      |
| `''` (string vazia)    | `mapmode-nvo`   | Normal, Visual, Select, Operator-pending | `:map`                    |
| `'n'`                  | `mapmode-n`     | Normal                                   | `:nmap`                   |
| `'v'`                  | `mapmode-v`     | Visual and Select                        | `:vmap`                   |
| `'s'`                  | `mapmode-s`     | Select                                   | `:smap`                   |
| `'x'`                  | `mapmode-x`     | Visual                                   | `:xmap`                   |
| `'o'`                  | `mapmode-o`     | Operator-pending                         | `:omap`                   |
| `'!'`                  | `mapmode-ic`    | Insert and Command-line                  | `:map!`                   |
| `'i'`                  | `mapmode-i`     | Insert                                   | `:imap`                   |
| `'l'`                  | `mapmode-l`     | Insert, Command-line, Lang-Arg           | `:lmap`                   |
| `'c'`                  | `mapmode-c`     | Command-line                             | `:cmap`                   |
| `'t'`                  | `mapmode-t`     | Terminal                                 | `:tmap`                   |

O segundo argumento é uma string contendo o lado esquerdo do mapeamento (a chave ou conjunto de chaves que acionam o comando definido no mapeamento). Uma string vazia é equivalente a `<Nop>`, que desabilita uma tecla.

O terceiro argumento é uma string contendo o lado direito do mapeamento (o comando a ser executado).
 
O argumento final é uma tabela contendo opções booleanas para o mapeamento conforme definido em `:help :map-arguments` (incluindo `noremap` e excluindo `buffer`).

Os mapeamentos locais do buffer também levam um número do buffer como primeiro argumento (`0` define o mapeamento para o buffer atual).

```lua
vim.api.nvim_set_keymap('n', '<leader><Space>', ':set hlsearch!<CR>', { noremap = true, silent = true })
-- :nnoremap <silent> <leader><Space> :set hlsearch<CR>

vim.api.nvim_buf_set_keymap(0, '', 'cc', 'line(".") == 1 ? "cc" : "ggcc"', { noremap = true, expr = true })
-- :noremap <buffer> <expr> cc line('.') == 1 ? 'cc' : 'ggcc'
```

`vim.api.nvim_get_keymap()` pega uma string contendo o nome curto do modo para o qual você deseja a lista de mapeamentos (veja a tabela acima). O valor de retorno é uma tabela contendo todos os mapeamentos globais para o modo.

```lua
print(vim.inspect(vim.api.nvim_get_keymap('n')))
-- :verbose nmap
```

`vim.api.nvim_buf_get_keymap()` recebe um número de buffer adicional como seu primeiro argumento (`0` obterá mapeamentos para o buffer atual)

```lua
print(vim.inspect(vim.api.nvim_buf_get_keymap(0, 'i')))
-- :verbose imap <buffer>
```

`vim.api.nvim_del_keymap()` pega um modo e o lado esquerdo de um mapeamento.

```lua
vim.api.nvim_del_keymap('n', '<leader><Space>')
-- :nunmap <leader><Space>
```

Novamente, `vim.api.nvim_buf_del_keymap()`, recebe um número de buffer como seu primeiro argumento, com `0` representando o buffer atual.

```lua
vim.api.nvim_buf_del_keymap(0, 'i', '<Tab>')
-- :iunmap <buffer> <Tab>
```

## Definindo comandos de usuário

Atualmente não há interface para criar comandos de usuário em Lua. Mas está planejado:

- [Solicitação pull # 11613](https://github.com/neovim/neovim/pull/11613)

Por enquanto, é melhor você criar comandos no Vimscript.

## Definindo autocommands

Augroups e autocommands ainda não têm uma interface, mas está sendo trabalhada:

- [Solicitação pull # 12378](https://github.com/neovim/neovim/pull/12378)

Enquanto isso, você pode criar comandos automáticos no Vimscript ou usar [este wrapper de norcalli / nvim_utils](https://github.com/norcalli/nvim_utils/blob/master/lua/nvim_utils.lua#L554-L567)

## Definindo sintaxe/destaques

A sintaxe API ainda é um trabalho em andamento. Aqui estão algumas dicas:

- [Problema nº 9876](https://github.com/neovim/neovim/issues/9876)
- [tjdevries / colorbuddy.vim, uma biblioteca para criar esquemas de cores em Lua](https://github.com/tjdevries/colorbuddy.vim)
- `:help lua-treesitter`

## Dicas e recomendações gerais

**FAÇAM**:
- Hot-reloadings de módulos
- `vim.validate()`?
- Adicionar coisas sobre testes de unidade? Eu sei que o Neovim usa o framework [busted](https://olivinelabs.com/busted/), mas não sei como usá-lo para plug-ins
- Melhores Práticas? Não sou um mago da Lua, então não saberia
- Como usar pacotes LuaRocks ([wbthomason / packer.nvim](https://github.com/wbthomason/packer.nvim)?)

## Diversos

### vim.loop

`vim.loop` é o módulo que expõe a API LibUV. Alguns recursos:

- [Documentação oficial para LibUV](https://docs.libuv.org/en/v1.x/)
- [documentação Luv](https://github.com/luvit/luv/blob/master/docs.md)
- [teukka.tech - Usando LibUV em Neovim](https://teukka.tech/vimloop.html)

Veja também:
- `:help vim.loop`

### vim.lsp

`vim.lsp` é o módulo que controla o cliente LSP embutido. O repositório [neovim / nvim-lspconfig](https://github.com/neovim/nvim-lspconfig/) contém configurações padrão para servidores de idiomas populares.

Você também pode dar uma olhada nos plug-ins construídos em torno do cliente LSP:
- [nvim-lua / completed-nvim](https://github.com/nvim-lua/completion-nvim)
- [nvim-lua / diagnostic-nvim](https://github.com/nvim-lua/diagnostic-nvim)

Veja também:
- `:help lsp`

### vim.treesitter

`vim.treesitter` é o módulo que controla a integração da biblioteca [Tree-sitter](https://tree-sitter.github.io/tree-sitter/) no Neovim. Se você quiser saber mais sobre o Tree-sitter, pode se interessar por esta [apresentação (38:37)](https://www.youtube.com/watch?v=Jes3bD6P0To).

A organização [nvim-treesitter](https://github.com/nvim-treesitter/) hospeda vários plug-ins aproveitando a biblioteca.

Veja também:
- `:help lua-treesitter`


### Transpilers

Uma vantagem de usar Lua é que você não precisa realmente escrever código Lua! Há uma grande variedade de transpiladores disponíveis para o idioma.

- [Moonscript](https://moonscript.org/)

Provavelmente um dos transpiladores mais conhecidos para Lua. Adiciona muitos recursos convenientes, como classes, compreensões de lista ou literais de função. O plugin [svermeulen / nvim-moonmaker](https://github.com/svermeulen/nvim-moonmaker) permite que você escreva plugins Neovim e configuração diretamente no Moonscript.

- [Fennel](https://fennel-lang.org/)

Um lisp que compila para Lua. Você pode escrever configuração e plug-ins para Neovim no Fennel com o plug-in [Olical / aniseed](https://github.com/Olical/aniseed). Adicionalmente, o plugin [Olical/conjure](https://github.com/Olical/conjure) disponibiliza um ambiente interativo de desenvlvimento que da suporte a Fennel (além de outras linguagens).

Outros projetos interessantes:
- [TypeScriptToLua/TypeScriptToLua](https://github.com/TypeScriptToLua/TypeScriptToLua)
- [teal-language/tl](https://github.com/teal-language/tl)
- [Haxe](https://haxe.org/)
- [SwadicalRag/wasm2lua](https://github.com/SwadicalRag/wasm2lua)
- [hengestone/lua-languages](https://github.com/hengestone/lua-languages)
