# Simularor de emergencia protecao em massa
Um simulador de emergência que age como treinamento para situações de perigo real. 
import heapq
import math
import random
import time
from dataclasses import dataclass, field
from typing import List, Tuple

@dataclass(order=True)
class Ocorrencia:
    prioridade: int  # 1 = Crítica (Alta), 2 = Urgente, 3 = Leve
    tempo_chegada: float
    id: int = field(compare=False)
    tipo: str = field(compare=False)
    localizacao: Tuple[int, int] = field(compare=False)

@dataclass
class Viatura:
    id: str
    tipo: str  # Ex: 'Ambulancia', 'Bombeiros', 'Policia'
    localizacao: Tuple[int, int]
    disponivel: bool = True

class CentralEmergencia:
    def __init__(self):
        self.fila_ocorrencias: List[Ocorrencia] = []
        self.viaturas: List[Viatura] = []
        self.contador_id = 1

    def adicionar_viatura(self, viatura: Viatura):
        self.viaturas.append(viatura)

    def registrar_chamado(self, tipo: str, prioridade: int, localizacao: Tuple[int, int]):
        ocorrencia = Ocorrencia(
            prioridade=prioridade,
            tempo_chegada=time.time(),
            id=self.contador_id,
            tipo=tipo,
            localizacao=localizacao
        )
        heapq.heappush(self.fila_ocorrencias, ocorrencia)
        self.contador_id += 1
        print(f"[CHAMADO #{ocorrencia.id}] {tipo} (Prioridade {prioridade}) em {localizacao}")

    def calcular_distancia(self, p1: Tuple[int, int], p2: Tuple[int, int]) -> float:
        return math.hypot(p1[0] - p2[0], p1[1] - p2[1])

    def despachar(self):
        if not self.fila_ocorrencias:
            return

        # Pega a ocorrência de maior prioridade (menor número)
        ocorrencia = heapq.heappop(self.fila_ocorrencias)
        
        # Filtra viaturas disponíveis
        disponiveis = [v for v in self.viaturas if v.disponivel]
        
        if not disponiveis:
            print(f"[ALERTA] Sem viaturas para o Chamado #{ocorrencia.id}. Retornando à fila.")
            heapq.heappush(self.fila_ocorrencias, ocorrencia)
            return

        # Escolhe a viatura mais próxima
        melhor_viatura = min(
            disponiveis, 
            key=lambda v: self.calcular_distancia(v.localizacao, ocorrencia.localizacao)
        )
        
        distancia = self.calcular_distancia(melhor_viatura.localizacao, ocorrencia.localizacao)
        melhor_viatura.disponivel = False
        melhor_viatura.localizacao = ocorrencia.localizacao
        
        print(f" -> [DESPACHO] {melhor_viatura.id} ({melhor_viatura.tipo}) enviada ao Chamado #{ocorrencia.id} | Distância: {distancia:.1f} km")

# --- Execução de Teste ---
if __name__ == "__main__":
    central = CentralEmergencia()
    
    # Criando frota inicial (coordenadas X, Y em um mapa 100x100)
    central.adicionar_viatura(Viatura("SAMU-01", "Ambulancia", (10, 15)))
    central.adicionar_viatura(Viatura("SAMU-02", "Ambulancia", (80, 75)))
    central.adicionar_viatura(Viatura("ABTS-01", "Bombeiros", (50, 50)))

    # Simulando entrada de chamados
    central.registrar_chamado("Incêndio Residencial", prioridade=1, localizacao=(55, 48))
    central.registrar_chamado("Queda de Moto", prioridade=2, localizacao=(12, 20))
    central.registrar_chamado("Parada Cardíaca", prioridade=1, localizacao=(78, 70))

    print("\n--- Processando Despachos ---")
    for _ in range(3):
        central.despachar()
