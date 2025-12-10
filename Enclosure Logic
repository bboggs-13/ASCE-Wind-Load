
from dataclasses import dataclass
from typing import Literal, Tuple

Enclosure = Literal['open','enclosed','partially_enclosed','partially_open']

@dataclass
class Openings:
    gross_N: float; open_N: float
    gross_E: float; open_E: float
    gross_S: float; open_S: float
    gross_W: float; open_W: float
    gross_Roof: float; open_Roof: float

    def each_wall_80pct_open(self) -> bool:
        return all(o >= 0.8*g for o,g in [
            (self.open_N, self.gross_N),
            (self.open_E, self.gross_E),
            (self.open_S, self.gross_S),
            (self.open_W, self.gross_W)
        ])

    def classify(self) -> Tuple[Enclosure, str]:
        if self.each_wall_80pct_open():
            return 'open', 'Each wall ≥80% open → Open building.'
        walls = {'N': (self.open_N, self.gross_N), 'E': (self.open_E, self.gross_E),
                 'S': (self.open_S, self.gross_S), 'W': (self.open_W, self.gross_W)}
        for key,(Ao,Ag) in walls.items():
            Aoi = sum(v[0] for k,v in walls.items() if k!=key) + self.open_Roof
            Agi = sum(v[1] for k,v in walls.items() if k!=key) + self.gross_Roof
            if Ao > 1.10*Aoi and Ao > min(4.0, 0.01*Ag) and (Aoi/Agi) <= 0.20:
                return 'partially_enclosed', f'Dominant opening on {key} >10% others; >4 ft² or >1% of wall; others ≤20%.'
        return 'enclosed', 'Not open/partially enclosed → Enclosed (override to partially_open if applicable).'
