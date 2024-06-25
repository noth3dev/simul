# simul

import matplotlib.pyplot as plt
import networkx as nx
import random
import numpy as np

# 파라미터 설정
initial_distribution = 5  # 초기 배포 영상 수
distribution_rate = 0.4  # 배포 확률
block_rate = 0.05  # 차단 확률
iterations = 3000  # 시뮬레이션 반복 횟수
income_per_view = 5  # 한 영상당 수익
loss_per_piracy = 50  # 한 불법 복제당 손실
watch_rate = 0.7  # 시청 확률
detection_delay = 3  # 불법 복제 탐지 지연 (반복 횟수)
viral_effect_rate = 2.0  # 바이럴 효과 증가율
legal_cost_per_action = 200  # 법적 대응 비용
ad_revenue_per_view = 1  # 광고 수익

# 시뮬레이션 초기화
total_income = 0
total_loss = 0
total_legal_costs = 0
total_ad_revenue = 0
video_status = {i: "distributed" for i in range(initial_distribution)}
video_graph = nx.DiGraph()
for i in range(initial_distribution):
    video_graph.add_node(i)

total_videos = [initial_distribution]
total_incomes = [0]
total_losses = [0]
total_legal_costs_list = [0]
total_ad_revenue_list = [0]

initial_income = initial_distribution * income_per_view
total_income += initial_income
total_incomes[0] = initial_income
for i in range(iterations):
    new_videos = 0
    blocked_videos = 0
    viral_boost = 1 if i < detection_delay else viral_effect_rate
    
    for video in list(video_status.keys()):
        if video_status[video] == "distributed":
            if random.random() < distribution_rate * viral_boost:
                new_video_id = len(video_status)
                video_status[new_video_id] = "distributed"
                video_graph.add_edge(video, new_video_id)
                new_videos += 1
                total_loss += loss_per_piracy
                
                if random.random() < watch_rate:
                    total_ad_revenue += ad_revenue_per_view

            if random.random() < block_rate:
                video_status[video] = "blocked"
                blocked_videos += 1
                total_legal_costs += legal_cost_per_action

    total_income += income_per_view * new_videos
    print(f"Iteration {i+1}: {new_videos} new videos, {blocked_videos} blocked videos")
    
    total_videos.append(len(video_status))
    total_incomes.append(total_income)
    total_losses.append(total_loss)
    total_legal_costs_list.append(total_legal_costs)
    total_ad_revenue_list.append(total_ad_revenue)

plt.figure(figsize=(14, 10))

plt.plot(range(iterations + 1), total_videos, marker='o', linestyle='-', color='b', label='Total Videos')
plt.plot(range(iterations + 1), total_incomes, marker='o', linestyle='-', color='g', label='Total Income')
plt.plot(range(iterations + 1), total_losses, marker='o', linestyle='-', color='r', label='Total Loss')
plt.plot(range(iterations + 1), total_legal_costs_list, marker='o', linestyle='-', color='purple', label='Total Legal Costs')
plt.plot(range(iterations + 1), total_ad_revenue_list, marker='o', linestyle='-', color='orange', label='Total Ad Revenue')

for i in range(iterations + 1):
    plt.text(i, total_videos[i], f'{total_videos[i]}', fontsize=9, ha='right', color='b')
    plt.text(i, total_incomes[i], f'${total_incomes[i]}', fontsize=9, ha='right', color='g')
    plt.text(i, total_losses[i], f'${total_losses[i]}', fontsize=9, ha='right', color='r')
    plt.text(i, total_legal_costs_list[i], f'${total_legal_costs_list[i]}', fontsize=9, ha='right', color='purple')
    plt.text(i, total_ad_revenue_list[i], f'${total_ad_revenue_list[i]}', fontsize=9, ha='right', color='orange')

plt.title('Total Videos, Income, Loss, Legal Costs, and Ad Revenue Over Time')
plt.xlabel('Iteration')
plt.ylabel('Amount')
plt.legend()
plt.grid(True)
plt.show()

print(f"Total income: ${total_income}")
print(f"Total loss: ${total_loss}")
print(f"Total legal costs: ${total_legal_costs}")
print(f"Total ad revenue: ${total_ad_revenue}")
