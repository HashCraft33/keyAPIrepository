# keyAPIrepository
import requests
import streamlit as st
import pandas as pd
import numpy as np
from functools import wraps
import matplotlib as mt

# Decorador para log de chamada de função
def log_call(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        st.write(f"Chamando a função: {func.__name__}")
        return func(*args, **kwargs)
    return wrapper


def log_call2(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        st.write(f"[LOG]: Executando {func.__name__}")
        return func(*args, **kwargs)
    return wrapper


# Classe que faz requisições para a API
class Get_Requests_API:
    def __init__(self, key_pix=None, key_register=None, key_ddd=None):
        self.key_pix = key_pix or "https://brasilapi.com.br/api/pix/v1/participants"
        self.key_register = key_register or "https://brasilapi.com.br/api/registrobr/v1/{domain}"
        self.key_ddd = key_ddd or "https://brasilapi.com.br/api/ddd/v1/11"  # Exemplo: DDD 11

    def get_pix_data(self):
        response = requests.get(self.key_pix)
        return response.json() if response.status_code == 200 else {"error": "Erro ao acessar a API PIX"}

    def get_register_data(self, domain):
        url = self.key_register.format(domain=domain)
        response = requests.get(url)
        return response.json() if response.status_code == 200 else {"error": "Erro ao acessar a API Registro BR"}

    def get_register_ddd(self):
        response = requests.get(self.key_ddd)
        return response.json() if response.status_code == 200 else {"error": "Erro ao acessar a API DDD"}


# Classe para exibir os dados usando Streamlit
class ShowWebPage(Get_Requests_API):
    def __init__(self):
        super().__init__()

    @log_call
    def get_webwrite(self):
        st.write("Visualização da API PIX:")
        data = self.get_pix_data()
        st.json(data)

    @log_call2
    def get_register_and_pix_data(self):
        st.write("Visualização da API Registro BR:")
        data = self.get_register_data("google.com")
        st.json(data)

        st.write("Visualização da API PIX novamente:")
        pix = self.get_pix_data()
        st.json(pix)

    @log_call2
    def show_ddd_data(self):
        st.write("Visualização da API DDD:")
        ddd_data = self.get_register_ddd()
        st.json(ddd_data)


class ShowCondicionalPage(ShowWebPage):
    def __init__(self):
        super().__init__()
    def searchShow(self,condicional_mostrar_dados):
        st.write(f"Buscando por: {condicional_mostrar_dados}")
        dataTime = self.get_pix_data()
        if "error" in dataTime:
            st.write(f"Erro: {dataTime['error']}")
            return 
        else:
            for i in range(len(dataTime['@log_call'])):
                for j in range(len(dataTime['@log_call2'])):
                    st.write(f"{dataTime['@log_call']}")
                    st.write(f"{dataTime['@log_call2']}")
    
# Execução principal
if __name__ == "__main__":
    st.title("Consulta de APIs - BrasilAPI")

    #page = ShowWebPage()
    condicional_mostrar_dados = st.text_input("Digite oque voce quer buscar: ")
    if condicional_mostrar_dados:
        page = ShowWebPage()
        page.seachShow(condicional_mostrar_dados)

    else:
        st.write("Digite algo para buscar na API")
        page = ShowCondicionalPage()
        page.searchShow(condicional_mostrar_dados)

    # Exemplo de dataframe para teste
        st.write("Exemplo de dados formatados pela API:")
 
    
    # Chamadas das funções
        page.get_webwrite()
        page.get_register_and_pix_data()
        page.show_ddd_data()
