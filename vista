from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Input(BaseModel):
    a: float
    b: float


import matplotlib.pyplot as plt
import csv
import os
import pandas as pd
from datetime import datetime
import yfinance as yf
import requests
from pytz import timezone
from difflib import get_close_matches

class ClearviewVista:
    def __init__(self):
        print("\n" + "="*70)
        print(" CLEARVIEW CAPITAL - VISTA PRO".center(70))
        print(" Visual Inteligente de Seleção Técnica e Analítica".center(70))
        print("="*70 + "\n")
        
        self.historico = {}
        self.tickers_map = {}
        self.nomes_map = {}
        self.carregar_dados_csv('acoes-listadas-b3.csv')
        
        # Cache para melhorar performance
        self.cache = {}
        self.ultimo_ticker = None
        
        print(f"✅ Base de ações carregada com {len(self.tickers_map)} ativos")

    def obter_data_hora_internet(self):
        """Obtém data e hora atual de um servidor confiável na internet"""
        try:
            response = requests.get("http://worldtimeapi.org/api/timezone/America/Sao_Paulo", timeout=5)
            dados = response.json()
            datetime_str = dados['datetime']
            dt = datetime.fromisoformat(datetime_str[:-6])
            return dt.astimezone(timezone('America/Sao_Paulo')).strftime("%d/%m/%Y %H:%M:%S")
        except Exception as e:
            print(f"⚠️ Falha ao obter data da internet: {e}. Usando data local.")
            return datetime.now().strftime("%d/%m/%Y %H:%M:%S")

    def carregar_dados_csv(self, arquivo):
        try:
            with open(arquivo, mode='r', encoding='utf-8') as f:
                reader = csv.DictReader(f, delimiter=',')
                for row in reader:
                    ticker = row['Ticker'].strip()
                    nome = row['Nome'].strip()
                    
                    # Criar versões com e sem .SA
                    ticker_sa = ticker + '.SA'
                    
                    # Mapear ticker para nome
                    self.tickers_map[ticker] = nome
                    self.tickers_map[ticker_sa] = nome
                    
                    # Mapear nome para tickers
                    nome_upper = nome.upper()
                    if nome_upper not in self.nomes_map:
                        self.nomes_map[nome_upper] = []
                    self.nomes_map[nome_upper].append(ticker_sa)
                    
                    # Adicionar o próprio ticker (sem .SA) como chave para o nome
                    if ticker not in self.nomes_map:
                        self.nomes_map[ticker] = []
                    self.nomes_map[ticker].append(ticker_sa)
        except Exception as e:
            print(f"⚠️ Erro ao carregar arquivo: {e}")
            self.criar_base_fallback()

    def criar_base_fallback(self):
        acoes_fallback = {
            'PETR4': 'Petrobras',
            'PETR4.SA': 'Petrobras',
            'PETR3': 'Petrobras',
            'PETR3.SA': 'Petrobras',
            'VALE3': 'Vale',
            'VALE3.SA': 'Vale',
            'ITUB4': 'Itaú Unibanco',
            'ITUB4.SA': 'Itaú Unibanco',
            'BBDC4': 'Banco Bradesco',
            'BBDC4.SA': 'Banco Bradesco',
            'B3SA3': 'B3',
            'B3SA3.SA': 'B3',
            'ABEV3': 'Ambev',
            'ABEV3.SA': 'Ambev',
            'MGLU3': 'Magazine Luiza',
            'MGLU3.SA': 'Magazine Luiza',
            'BBAS3': 'Banco do Brasil',
            'BBAS3.SA': 'Banco do Brasil',
            'WEGE3': 'WEG',
            'WEGE3.SA': 'WEG',
            'SUZB3': 'Suzano',
            'SUZB3.SA': 'Suzano',
            'SAPR4': 'Sanepar',
            'SAPR4.SA': 'Sanepar',
            'SAPR3': 'Sanepar',
            'SAPR3.SA': 'Sanepar',
            'ITSA4': 'Itaúsa',
            'ITSA4.SA': 'Itaúsa',
            'RENT3': 'Localiza',
            'RENT3.SA': 'Localiza',
            'BOVA11': 'ETF BOVA11',
            'BOVA11.SA': 'ETF BOVA11'
        }
        
        self.tickers_map = acoes_fallback
        self.nomes_map = {}
        
        for ticker, nome in acoes_fallback.items():
            nome_upper = nome.upper()
            if nome_upper not in self.nomes_map:
                self.nomes_map[nome_upper] = []
            self.nomes_map[nome_upper].append(ticker)
            
            # Mapear também o próprio ticker para si mesmo
            if ticker not in self.nomes_map:
                self.nomes_map[ticker] = []
            self.nomes_map[ticker].append(ticker)

    def encontrar_correspondencia_aproximada(self, entrada):
        """Encontra correspondência aproximada para nomes de empresas ou tickers"""
        entrada = entrada.upper().strip()
        todas_chaves = list(self.nomes_map.keys())
        
        # Procurar correspondências próximas
        matches = get_close_matches(entrada, todas_chaves, n=3, cutoff=0.6)
        
        return matches

    def buscar_tickers_por_nome(self, entrada):
        entrada = entrada.upper().strip()
        # Busca direta por ticker ou nome
        if entrada in self.nomes_map:
            return sorted(self.nomes_map[entrada], 
                         key=lambda t: '4' if t.endswith('4') or t.endswith('4.SA') else 
                                      ('3' if t.endswith('3') or t.endswith('3.SA') else 'z'))
        return []

    def formatar_numero(self, valor):
        try:
            if valor is None or valor == '' or valor == 'N/A':
                return 'N/A'
            
            if isinstance(valor, str):
                # Remover caracteres não numéricos
                valor = ''.join(filter(lambda x: x.isdigit() or x in ['.', ',', '-'], valor))
                valor = valor.replace('.', '').replace(',', '.')
                valor = float(valor)
            
            if isinstance(valor, (int, float)):
                return f"{valor:.2f}"
            return str(valor)
        except:
            return str(valor)

    def formatar_percentual(self, valor):
        try:
            if valor is None or valor == '' or valor == 'N/A':
                return 'N/A'
                
            if isinstance(valor, str):
                # Remover o símbolo de percentual
                valor = valor.replace('%', '').strip()
                try:
                    valor = float(valor)
                except:
                    return 'N/A'
            
            # Garantir que o valor esteja em formato decimal
            if isinstance(valor, (int, float)):
                if valor > 20:
                    valor = valor / 100
                return f"{valor:.2f}%"
            return str(valor)
        except:
            return str(valor)

    def formatar_moeda(self, valor):
        try:
            if valor is None or valor == '' or valor == 'N/A':
                return 'N/A'
                
            if isinstance(valor, str):
                valor = float(valor.replace('.', '').replace(',', '.'))
                
            if valor >= 1_000_000_000:
                return f"R$ {valor/1_000_000_000:.2f} Bi"
            elif valor >= 1_000_000:
                return f"R$ {valor/1_000_000:.2f} Mi"
            else:
                return f"R$ {valor:,.2f}"
        except:
            return str(valor)

    def calcular_graham(self, lpa, vpa):
        try:
            if isinstance(lpa, str):
                lpa = float(lpa.replace('R$', '').replace(',', '.').strip())
            if isinstance(vpa, str):
                vpa = float(vpa.replace('R$', '').replace(',', '.').strip())
                
            if lpa <= 0 or vpa <= 0:
                return 'N/A (LPA ou VPA negativos)'
                
            valor_justo = (22.5 * lpa * vpa) ** 0.5
            return f"R$ {valor_justo:.2f}"
        except:
            return 'N/A'

    def buscar_indicadores_fallback(self, info):
        try:
            pe_ratio = info.get('trailingPE')
            pb_ratio = info.get('priceToBook')
            dividend_yield = info.get('dividendYield')
            market_cap = info.get('marketCap')
            ev_ebitda = info.get('enterpriseToEbitda')
            roe = info.get('returnOnEquity')
            profit_margins = info.get('profitMargins')
            eps = info.get('trailingEps')
            book_value = info.get('bookValue')
            
            if dividend_yield is not None:
                dividend_yield = dividend_yield * 100
                if dividend_yield > 20:
                    dividend_yield = dividend_yield / 100
            
            if roe is not None:
                roe = roe * 100
            if profit_margins is not None:
                profit_margins = profit_margins * 100
            
            graham_value = 'N/A'
            if eps is not None and book_value is not None and eps > 0 and book_value > 0:
                graham_value = self.calcular_graham(eps, book_value)
            
            resultados = {
                'P/L': self.formatar_numero(pe_ratio),
                'P/VP': self.formatar_numero(pb_ratio),
                'Dividend Yield': self.formatar_percentual(dividend_yield),
                'Valor de Mercado': self.formatar_moeda(market_cap),
                'EV/EBITDA': self.formatar_numero(ev_ebitda),
                'ROE': self.formatar_percentual(roe),
                'Marg. Líquida': self.formatar_percentual(profit_margins),
                'LPA': self.formatar_numero(eps),
                'VPA': self.formatar_numero(book_value),
                'Valor Graham': graham_value
            }
            
            # Adicionar mais indicadores
            if 'enterpriseValue' in info:
                resultados['Enterprise Value'] = self.formatar_moeda(info.get('enterpriseValue'))
            if 'forwardPE' in info:
                resultados['P/L Futuro'] = self.formatar_numero(info.get('forwardPE'))
            if 'pegRatio' in info:
                resultados['PEG Ratio'] = self.formatar_numero(info.get('pegRatio'))
            if 'priceToSalesTrailing12Months' in info:
                resultados['P/Vendas'] = self.formatar_numero(info.get('priceToSalesTrailing12Months'))
            if 'returnOnAssets' in info:
                resultados['ROA'] = self.formatar_percentual(info.get('returnOnAssets') * 100 if info.get('returnOnAssets') else None)
            if 'debtToEquity' in info:
                resultados['Dívida/Patrimônio'] = self.formatar_numero(info.get('debtToEquity'))
            if 'currentRatio' in info:
                resultados['Liquidez Corrente'] = self.formatar_numero(info.get('currentRatio'))
            if 'quickRatio' in info:
                resultados['Liquidez Seca'] = self.formatar_numero(info.get('quickRatio'))
            if 'totalCash' in info:
                resultados['Caixa Total'] = self.formatar_moeda(info.get('totalCash'))
            if 'totalDebt' in info:
                resultados['Dívida Total'] = self.formatar_moeda(info.get('totalDebt'))
            if 'totalRevenue' in info:
                resultados['Receita Total'] = self.formatar_moeda(info.get('totalRevenue'))
            
            return resultados
        except Exception as e:
            print(f"⚠️ Erro no fallback de indicadores: {str(e)}")
            return {}

    def buscar_cotacao_completa(self, entrada):
        try:
            # Verificar se é um ticker válido
            ticker = entrada.upper().strip()
            if not ticker.endswith('.SA'):
                ticker_sa = ticker + '.SA'
                if ticker_sa in self.tickers_map:
                    ticker = ticker_sa
            
            # Verificar cache primeiro
            if ticker in self.cache:
                return self.cache[ticker]
                
            acao = yf.Ticker(ticker)
            info = acao.info
            hist = acao.history(period="1d")
            
            if hist.empty:
                return None
            
            price = info.get('regularMarketPrice', hist['Close'].iloc[-1])
            previous_close = info.get('regularMarketPreviousClose', hist['Close'].iloc[-1])
            change = price - previous_close
            percent_change = (change / previous_close) * 100 if previous_close else 0
            
            indicadores_avancados = self.buscar_indicadores_fallback(info)
            
            dados = {
                'symbol': ticker,
                'name': self.tickers_map.get(ticker, info.get('longName', ticker)),
                'price': price,
                'open': hist['Open'].iloc[-1] if not hist['Open'].empty else price,
                'high': hist['High'].iloc[-1] if not hist['High'].empty else price,
                'low': hist['Low'].iloc[-1] if not hist['Low'].empty else price,
                'volume': hist['Volume'].iloc[-1] if not hist['Volume'].empty else 0,
                'previous_close': previous_close,
                'change': change,
                'percent_change': f"{percent_change:.2f}%",
                'indicadores': indicadores_avancados,
                'data_consulta': self.obter_data_hora_internet()  # Usa data da internet
            }
            
            # Salvar no cache
            self.cache[ticker] = dados
            return dados
            
        except Exception as e:
            print(f"\nErro ao buscar dados: {str(e)}")
            return None

    def mostrar_analise_completa(self, ticker, dados):
        if not dados:
            print("❌ Não foi possível obter dados para esta ação")
            return
            
        print("\n" + "="*70)
        ticker_display = ticker.replace('.SA', '') if ticker.endswith('.SA') else ticker
        print(f"📊 CLEARVIEW VISTA: {ticker_display} - {dados['name']}".center(70))
        print(f"📅 Data da consulta: {dados['data_consulta']}".center(70))
        print("="*70)
        
        print("\n💹 COTAÇÃO")
        print(f"💰 Preço Atual: R$ {dados['price']:.2f}")
        print(f"📈 Variação: {dados['percent_change']} (R$ {dados['change']:.2f})")
        print(f"🔄 Volume: {dados['volume']:,}")
        
        print("\n📅 INFORMAÇÕES DO DIA")
        print(f"🔹 Abertura: R$ {dados['open']:.2f}")
        print(f"🔹 Mínima: R$ {dados['low']:.2f}")
        print(f"🔹 Máxima: R$ {dados['high']:.2f}")
        print(f"🔹 Fech. anterior: R$ {dados['previous_close']:.2f}")
        
        print("\n📊 INDICADORES FUNDAMENTALISTAS")
        indicadores = dados['indicadores']
        
        ordem_indicadores = [
            'P/L', 'P/VP', 'Dividend Yield', 'Valor de Mercado', 'EV/EBITDA', 
            'LPA', 'VPA', 'ROE', 'Valor Graham', 'Marg. Líquida',
            'ROA', 'Dívida/Patrimônio', 'Liquidez Corrente',
            'P/L Futuro', 'P/Vendas', 'PEG Ratio', 'Enterprise Value',
            'Caixa Total', 'Dívida Total', 'Receita Total'
        ]
        
        for indicador in ordem_indicadores:
            if indicador in indicadores and indicadores[indicador] != 'N/A':
                valor = indicadores[indicador]
                print(f"🔸 {indicador}: {valor}")
        
        for indicador, valor in indicadores.items():
            if indicador not in ordem_indicadores and valor != 'N/A':
                print(f"🔸 {indicador}: {valor}")
        
        print("="*70)
        self.ultimo_ticker = ticker

    def gerar_grafico(self, periodo='6mo'):
        if not self.ultimo_ticker:
            print("❌ Consulte uma ação primeiro")
            return
            
        ticker = self.ultimo_ticker
        nome = self.tickers_map.get(ticker, ticker)
        
        try:
            print(f"\n⏳ Gerando gráfico para {ticker}...")
            acao = yf.Ticker(ticker)
            hist = acao.history(period=periodo)
            
            if hist.empty:
                print("❌ Dados históricos indisponíveis")
                return
                
            plt.figure(figsize=(14, 7))
            plt.plot(hist.index, hist['Close'], 'b-', linewidth=1.5)
            plt.title(f'{ticker.replace(".SA", "")} - {nome}', fontsize=16)
            plt.xlabel('Data', fontsize=12)
            plt.ylabel('Preço (R$)', fontsize=12)
            plt.grid(True, linestyle='--', alpha=0.7)
            plt.tight_layout()
            
            ultimo_preco = hist['Close'].iloc[-1]
            plt.annotate(f'R$ {ultimo_preco:.2f}', 
                         xy=(hist.index[-1], ultimo_preco),
                         xytext=(10, -20),
                         textcoords='offset points',
                         arrowprops=dict(arrowstyle='->', color='black'))
            
            plt.show()
            
        except Exception as e:
            print(f"❌ Erro ao gerar gráfico: {str(e)}")

def main():
    plataforma = ClearviewVista()
    
    print("\n🔍 Digite o ticker (ex: PETR4) ou nome da empresa (ex: Petrobras)")
    print("   Comandos disponíveis:")
    print("   - 'grafico': Gera gráfico da última ação")
    print("   - 'grafico 1y': Gráfico de 1 ano")
    print("   - 'sair': Encerra o programa")

    while True:
        comando = input("\nClearview Vista >> ").strip()
        
        if comando.lower() in ['sair', 'exit', 'quit']:
            print("\n" + "="*70)
            print(" Análise encerrada ".center(70))
            print("="*70)
            break
            
        if not comando:
            continue
            
        # Comando para gerar gráfico
        if comando.lower().startswith('grafico'):
            partes = comando.split()
            periodo = partes[1] if len(partes) > 1 else '6mo'
            plataforma.gerar_grafico(periodo)
            continue
            
        # Primeiro tentar buscar como ticker (com ou sem .SA)
        ticker_input = comando.upper().strip()
        if ticker_input in plataforma.tickers_map:
            dados = plataforma.buscar_cotacao_completa(ticker_input)
            if dados:
                plataforma.mostrar_analise_completa(ticker_input, dados)
                continue
                
        # Se não encontrou, tentar adicionar .SA se não tiver
        if not ticker_input.endswith('.SA'):
            ticker_sa = ticker_input + '.SA'
            if ticker_sa in plataforma.tickers_map:
                dados = plataforma.buscar_cotacao_completa(ticker_sa)
                if dados:
                    plataforma.mostrar_analise_completa(ticker_sa, dados)
                    continue
        
        # Se não for ticker, buscar por nome
        tickers = plataforma.buscar_tickers_por_nome(comando)
        
        if not tickers:
            # Tolerância a erros de digitação - buscar correspondência aproximada
            correspondencias = plataforma.encontrar_correspondencia_aproximada(comando)
            
            if correspondencias:
                print("\n🔍 Talvez você quis dizer:")
                for i, corr in enumerate(correspondencias, 1):
                    print(f"{i} - {corr}")
                
                try:
                    escolha = int(input("Escolha o número correspondente (ou 0 para cancelar): "))
                    if 1 <= escolha <= len(correspondencias):
                        comando_corrigido = correspondencias[escolha-1]
                        # Tentar novamente com o comando corrigido
                        if comando_corrigido in plataforma.nomes_map:
                            tickers = plataforma.buscar_tickers_por_nome(comando_corrigido)
                        else:
                            # Se não for um nome, tratar como ticker
                            dados = plataforma.buscar_cotacao_completa(comando_corrigido)
                            if dados:
                                plataforma.mostrar_analise_completa(comando_corrigido, dados)
                                continue
                    elif escolha != 0:
                        print("❌ Escolha inválida")
                        continue
                except:
                    print("❌ Entrada inválida")
                    continue
        
        if tickers:
            print(f"\n🔍 Ações disponíveis para '{comando}':")
            print("-"*70)
            print("TIPO | TICKER   | DESCRIÇÃO")
            print("-"*70)
            
            acoes_pn = []
            acoes_on = []
            
            for t in tickers:
                tipo = "PN" if t.endswith('4') or t.endswith('4.SA') else "ON" if t.endswith('3') or t.endswith('3.SA') else "OUT"
                descricao = plataforma.tickers_map[t]
                
                if tipo == "PN":
                    acoes_pn.append((t, descricao))
                elif tipo == "ON":
                    acoes_on.append((t, descricao))
                else:
                    print(f"{tipo}  | {t.replace('.SA', ''):<8} | {descricao}")
            
            for t, desc in acoes_pn:
                print(f"PN   | {t.replace('.SA', ''):<8} | {desc}")
            
            for t, desc in acoes_on:
                print(f"ON   | {t.replace('.SA', ''):<8} | {desc}")
            
            # Seleção automática se só houver uma opção
            if len(acoes_pn) + len(acoes_on) == 1:
                ticker = (acoes_pn + acoes_on)[0][0]
                print(f"\n✅ Selecionada única opção: {ticker.replace('.SA', '')}")
                dados = plataforma.buscar_cotacao_completa(ticker)
                if dados:
                    plataforma.mostrar_analise_completa(ticker, dados)
            else:
                print("\n💡 Escolha o tipo de ação:")
                print("   4 para PN (Preferenciais)")
                print("   3 para ON (Ordinárias)")
                
                escolha = input("Tipo de ação (3/4): ").strip()
                
                if escolha == '4':
                    tickers_selecionados = acoes_pn
                elif escolha == '3':
                    tickers_selecionados = acoes_on
                else:
                    print("❌ Opção inválida")
                    continue
                    
                if not tickers_selecionados:
                    print("❌ Nenhuma ação deste tipo disponível")
                    continue
                    
                # Seleção automática se só houver uma do tipo
                if len(tickers_selecionados) == 1:
                    ticker = tickers_selecionados[0][0]
                    dados = plataforma.buscar_cotacao_completa(ticker)
                    if dados:
                        plataforma.mostrar_analise_completa(ticker, dados)
                else:
                    print("\n🔍 Ações disponíveis deste tipo:")
                    for i, (t, desc) in enumerate(tickers_selecionados, 1):
                        print(f"{i} - {t.replace('.SA', '')}: {desc}")
                    
                    escolha_num = input("\nEscolha o número da ação: ").strip()
                    try:
                        idx = int(escolha_num) - 1
                        if 0 <= idx < len(tickers_selecionados):
                            ticker = tickers_selecionados[idx][0]
                            dados = plataforma.buscar_cotacao_completa(ticker)
                            if dados:
                                plataforma.mostrar_analise_completa(ticker, dados)
                        else:
                            print("❌ Número inválido")
                            continue
                    except:
                        print("❌ Entrada inválida")
                        continue
        else:
            print(f"❌ Não foi possível encontrar dados para '{comando}'")

if __name__ == "__main__":
    try:
        import yfinance as yf
        main()
    except ImportError:
        print("\n⚠️ A biblioteca yfinance não está instalada.")
        print("   Instale-a com o comando: pip install yfinance")
    except Exception as e:
        print(f"\n❌ Erro: {str(e)}")
