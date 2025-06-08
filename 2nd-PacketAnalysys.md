sudo apt install -y python3-pip powertop plantuml
pip3 install stable-baselines3 gym numpy pandas matplotlib

mkdir -p ~/oran_mininet/{src,data,results}

اسکریپت توپولوژی:

فایل src/oran_topology.py:

from mininet.net import Mininet
from mininet.node import Host, OVSSwitch, Controller
from mininet.link import TCLink
from mininet.cli import CLI
from mininet.log import setLogLevel

def oran_topology():
    net = Mininet(controller=Controller, switch=OVSSwitch, link=TCLink)
    
    c0 = net.addController('c0')
    ric = net.addHost('ric', ip='10.0.0.1')
    o_cu = net.addHost('o_cu', ip='10.0.0.2')
    o_du = net.addHost('o_du', ip='10.0.0.3')
    o_ru1 = net.addHost('o_ru1', ip='10.0.0.4')
    o_ru2 = net.addHost('o_ru2', ip='10.0.0.5')
    s1 = net.addSwitch('s1')
    
    net.addLink(o_ru1, s1, bw=100, delay='5ms')
    net.addLink(o_ru2, s1, bw=100, delay='5ms')
    net.addLink(s1, o_du, bw=1000, delay='2ms')
    net.addLink(o_du, o_cu, bw=1000, delay='1ms')
    net.addLink(o_cu, ric, bw=1000, delay='1ms')
    
    net.start()
    CLI(net)
    net.stop()

if __name__ == '__main__':
    setLogLevel('info')
    oran_topology()
    
=================================
run script 

pingall 

گام 3: شبیه‌سازی O-RAN و تولید داده‌ها
هدف: تولید داده‌های انرژی و تأخیر.

اسکریپت شبیه‌سازی:
فایل src/oran_sim.py:


import random
import pandas as pd
import time

def simulate_oran(nodes, duration):
    data = {"Time": [], "Node": [], "Energy": [], "Latency": []}
    
    for t in range(duration):
        for node in nodes:
            if "O-RU" in node:
                energy = random.uniform(80, 120)
            elif "O-DU" in node:
                energy = random.uniform(40, 60)
            else:
                energy = random.uniform(10, 20)
            latency = random.uniform(5, 20)
            data["Time"].append(t)
            data["Node"].append(node)
            data["Energy"].append(energy)
            data["Latency"].append(latency)
        time.sleep(0.1)
    
    df = pd.DataFrame(data)
    df.to_csv("data/oran_data.csv", index=False)

if __name__ == "__main__":
    nodes = ["O-RU1", "O-RU2", "O-DU", "O-CU", "RIC"]
    simulate_oran(nodes, duration=100)
    
==================================
python3 src/oran_sim.py
 run on mininet container : 
sudo powertop --csv=data/power_data.csv

===========================================================
گام 4: پیاده‌سازی F-DDPG : https://grok.com/chat/44ae95c7-409f-4634-8f1f-81bb39136b74