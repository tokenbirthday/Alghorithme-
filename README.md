```python
import pandas as pd
import numpy as np
import ta  # Assurez-vous d'avoir installé la bibliothèque ta

# Charger les données historiques
data = pd.read_csv('historical_data.csv')

# Calcul des indicateurs techniques
data['RSI'] = ta.momentum.RSIIndicator(data['close'], window=14).rsi()
data['MACD'] = ta.trend.MACD(data['close']).macd()
data['MACD_signal'] = ta.trend.MACD(data['close']).macd_signal()

# Stratégie de trading
data['signal'] = 0
data['signal'][14:] = np.where((data['RSI'][14:] < 30) & (data['MACD'][14:] > data['MACD_signal'][14:]), 1, 0)  # Achat
data['signal'][14:] = np.where((data['RSI'][14:] > 70) & (data['MACD'][14:] < data['MACD_signal'][14:]), -1, data['signal'][14:])  # Vente

# Simuler les transactions
data['position'] = data['signal'].replace(to_replace=0, method='ffill')  # Maintenir la position
data['daily_return'] = data['close'].pct_change()  # Rendement quotidien
data['strategy_return'] = data['daily_return'] * data['position'].shift(1)  # Rendement de la stratégie

# Gestion des risques : définir un stop loss et un take profit
stop_loss = 0.02  # 2% de perte maximale
take_profit = 0.05  # 5% de gain maximal

data['cumulative_return'] = (1 + data['strategy_return']).cumprod()

# Appliquer le stop loss et le take profit
data['final_return'] = np.where(data['strategy_return'] < -stop_loss, -stop_loss, data['strategy_return'])
data['final_return'] = np.where(data['strategy_return'] > take_profit, take_profit, data['final_return'])

# Calculer le rendement cumulatif final
data['cumulative_final_return'] = (1 + data['final_return']).cumprod()

# Afficher les résultats
print(data[['close', 'RSI', 'MACD', 'signal', 'cumulative_return', 'cumulative_final_return']])
```

Dans cette version :

1. Gestion des risques : J'ai ajouté un stop loss de 2% et un take profit de 5%. Cela signifie que si la perte d'une transaction dépasse 2%, elle sera arrêtée, et si le gain atteint 5%, la transaction sera également arrêtée.
2. Rendement final : Le rendement final prend en compte ces limites de perte et de gain.

Assure-toi de bien tester cette stratégie avec des données historiques avant de l'appliquer dans un environnement réel. Si tu as d'autres questions ou si tu veux explorer des aspects spécifiques, fais-le moi savoir !
