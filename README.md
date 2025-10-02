# binance-trading-bot
A simple Binance Futures Testnet Trading Bot (Internship task)
import logging
import sys
from binance import Client
from binance.exceptions import BinanceAPIException, BinanceOrderException

class BasicBot:
    def _init_(self, api_key, api_secret, testnet=True):
        self.client = Client(api_key, api_secret)
        if testnet:
            # Use Binance Futures Testnet base URL
            self.client.API_URL = 'https://testnet.binancefuture.com/fapi/v1'

        # Setup logging to both file and console
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s:%(levelname)s:%(message)s',
            handlers=[
                logging.FileHandler("trading_bot.log"),
                logging.StreamHandler(sys.stdout)
            ]
        )
        logging.info("BasicBot initialized.")

    def place_order(self, symbol, side, order_type, quantity, price=None):
        try:
            logging.info(f"Placing {order_type} {side} order for {quantity} {symbol} at price {price if price else 'Market Price'}")

            if order_type == 'MARKET':
                order = self.client.futures_create_order(
                    symbol=symbol,
                    side=side,
                    type=order_type,
                    quantity=quantity
                )
            elif order_type == 'LIMIT':
                if price is None:
                    logging.error("Price required for LIMIT orders.")
                    return None
                order = self.client.futures_create_order(
                    symbol=symbol,
                    side=side,
                    type=order_type,
                    quantity=quantity,
                    price=price,
                    timeInForce='GTC'
                )
            else:
                logging.error(f"Unsupported order type: {order_type}")
                return None

            logging.info(f"Order response: {order}")
            return order

        except BinanceAPIException as e:
            logging.error(f"Binance API Exception: {e.message}")
            return None
        except BinanceOrderException as e:
            logging.error(f"Binance Order Exception: {e.message}")
            return None
        except Exception as e:
            logging.error(f"Unexpected error: {str(e)}")
            return None


def get_user_input():
    print("=== Binance Futures Testnet Trading Bot ===")
    api_key = input("Enter your Binance Testnet API Key: ").strip()
    api_secret = input("Enter your Binance Testnet API Secret: ").strip()

    symbol = input("Enter trading pair symbol (e.g., BTCUSDT): ").strip().upper()
    while not symbol.isalnum() or len(symbol) < 6:
        symbol = input("Invalid symbol. Enter trading pair symbol (e.g., BTCUSDT): ").strip().upper()

    side = input("Order side (BUY/SELL): ").strip().upper()
    while side not in ['BUY', 'SELL']:
        side = input("Invalid input. Enter order side (BUY/SELL): ").strip().upper()

    order_type = input("Order type (MARKET/LIMIT): ").strip().upper()
    while order_type not in ['MARKET', 'LIMIT']:
        order_type = input("Invalid input. Enter order type (MARKET/LIMIT): ").strip().upper()

    quantity_str = input("Enter quantity (e.g., 0.001): ").strip()
    try:
        quantity = float(quantity_str)
        if quantity <= 0:
            raise ValueError
    except ValueError:
        print("Invalid quantity. Must be a positive number.")
        sys.exit(1)

    price = None
    if order_type == 'LIMIT':
        price_str = input("Enter price for limit order: ").strip()
        try:
            price = float(price_str)
            if price <= 0:
                raise ValueError
        except ValueError:
            print("Invalid price. Must be a positive number.")
            sys.exit(1)

    return api_key, api_secret, symbol, side, order_type, quantity, price


def main():
    api_key, api_secret, symbol, side, order_type, quantity, price = get_user_input()

    bot = BasicBot(api_key, api_secret, testnet=True)
    order = bot.place_order(symbol, side, order_type, quantity, price)

    if order:
        print("\nOrder placed successfully:")
        print(order)
    else:
        print("\nFailed to place order. Check the logs for details.")


if _name_ == "_main_":
    main()
