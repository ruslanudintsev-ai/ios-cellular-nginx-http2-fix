# iPhone não abre site HTTPS pelos dados móveis? Um caso real com Nginx e HTTP/2

**Idiomas:** [English](README.md) · [Русский](README.ru.md) · [Español](README.es.md) · [Português (Brasil)](README.pt-BR.md) · [简体中文](README.zh-CN.md)

**Status do caso:** depois de habilitar HTTP/2 nos servidores virtuais HTTPS de produção, o mesmo iPhone abriu o site imediatamente pela rede móvel, sem VPN. O resultado foi registrado em **10 de outubro de 2026**. Na publicação deste relato, ainda não haviam passado 24 horas; a estabilidade a longo prazo não estava comprovada.

Este estudo trata de [polishelp.ru](https://polishelp.ru/). É um relato técnico baseado nas observações do responsável pelo site, **não um experimento de laboratório**.

## Resumo

Durante a investigação, o Safari em iPhones às vezes não exibia o site usando dados móveis sem VPN. Pelo Wi-Fi ou com VPN, o acesso funcionava; também foi informado que Android pela rede móvel funcionava. As evidências foram inconsistentes: em algumas tentativas, a conexão parou **antes da requisição HTTP**; em outras, o HTML recebeu resposta **HTTP 200**, mas a página não apareceu corretamente.

A equipe experimentou outro servidor e outros nomes de host, HTML estático simplificado, JavaScript, gzip, prefetch do Next.js, configurações TLS e captura de pacotes. Nenhuma dessas abordagens resultou em uma correção confiável. Em produção, o Nginx negociava **HTTP/1.1**.

Primeiro, o HTTP/2 foi habilitado em um servidor virtual de diagnóstico. O responsável relatou que ele abriu imediatamente no iPhone afetado pela rede móvel, sem VPN. Depois, HTTP/2 foi habilitado nos servidores virtuais HTTPS do domínio principal e do `www`. A mesma URL de produção abriu imediatamente no mesmo iPhone e na mesma conexão móvel. O fallback para HTTP/1.1 foi preservado.

## O que a evidência permite concluir

A comparação antes e depois **no mesmo domínio e certificado de produção** é um indício forte de que a mudança para HTTP/2, ou sua interação com o caminho anterior via HTTP/1.1, foi relevante. É mais convincente do que uma comparação entre domínios diferentes.

Isso **não comprova um defeito específico no Safari, no Nginx, em TLS, em HTTP/1.1 ou na operadora**. Os resultados no iPhone foram relatados pelo responsável e não houve captura de pacotes sincronizada com a tentativa. As condições da rede e o estado do navegador também podem variar.

A ativação exigiu ainda um reload gracioso do Nginx, que pode ter alterado o estado das conexões. Uma única abertura bem-sucedida não isola completamente o efeito do HTTP/2 do efeito do reload. A causa técnica de baixo nível permanece desconhecida.

Na alteração de produção, a política TLS, as cifras, o código do aplicativo, o DNS e o conteúdo do site não foram modificados.

## Documentação técnica

Os documentos detalhados estão disponíveis em inglês e russo:

- [Linha do tempo (EN)](docs/timeline.md) · [RU](docs/timeline.ru.md)
- [Tentativas sem resultado conclusivo (EN)](docs/failed-attempts.md) · [RU](docs/failed-attempts.ru.md)
- [Método de diagnóstico (EN)](docs/diagnostics.md) · [RU](docs/diagnostics.ru.md)
- [Ativar HTTP/2 no Nginx e reverter a mudança (EN)](docs/nginx-http2.md) · [RU](docs/nginx-http2.ru.md)

## Cuidados ao reproduzir

**Não presuma que HTTP/2 seja uma solução universal.** Verifique a versão e o módulo HTTP/2 do Nginx, preserve a configuração anterior, execute `nginx -t`, altere apenas os servidores virtuais pretendidos e tenha um procedimento de rollback. Confirme que HTTP/2 foi de fato negociado, que HTTP/1.1 continua disponível e repita os testes no dispositivo e na rede afetados.

A diretiva `http2 on;` está disponível desde o Nginx **1.25.1**: [documentação oficial](https://nginx.org/en/docs/http/ngx_http_v2_module.html).

## Privacidade e licença

O relato não divulga endereços IP internos, logs brutos de clientes, caminhos administrativos privados, credenciais ou links para o acompanhamento interno do incidente. Não compartilhe capturas de tráfego com dados pessoais em issues públicas.

Documentação licenciada sob [CC BY 4.0](LICENSE).
