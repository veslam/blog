---
title: 一道题的两个解法
date: 2016-12-13 00:04:45
tags: [coding, OJ, priority queue, binary search]
---
HackerRank的HourOfCode2016题组，平均难度较低，然而有一道题把我难住了。
>[The great robbery](https://www.hackerrank.com/contests/hour-of-code-2016/challenges/the-great-robbery/copy-from/8049286)

>While going to winter vacation you found a beautiful frozen lake. You noticed something shiny in the middle of it. When you got closer you realized that there is a huge pile of gold bars on a little island.
>
Realizing that you are rich, you started dreaming about everything you could buy, but first of all you have to collect all the gold. Unfortunately, the ice on the lake is not strong enough and cannot support the weight of all the gold. However, you know that the lake can be represented by a 2 dimensional matrix where every cell of the matrix has a value corresponding to the maximum weight of gold that can be transported over that field.
>
You are given the coordinates of your current position and the coordinates of your destination. Find a route that enables you to take as much gold as possible and print that amount of gold. You can asume that the amount of gold on the island is always equal to the maximal amount of gold that can be on that tile.
>
Keep in mind that the path that enables you to take the highest amount of gold doesn't have to be the shortest path!

>__Input Format__
The first row contains two numbers __N__ and __M__, the height and width of the lake.
The second row contains two numbers __StartX__ and __StartY__, representing the starting coordinates.
The third row contains two numbers __DestinationX__ and __DestinationY__, representing the coordinates of the destination.
This is followed by __N__ rows containing __M__ numbers each, containing the values for the matrix representing the lake.
>
__Sample__
![Start和End之间最大路径宽度为5](https://github.com/veslam/ImagesForBlog/raw/master/res/20161213_01_HackerRank.png)

__题意__
四连通矩阵，求起始点到目标点之间，最大的路径容量。路径容量由途经元素中最小的容量决定。

__我的思路__
建等大矩阵max2start，每个元素的值表示由起始点到当前点的已知最大路径容量，初始化都为0。采用队列结构，每个循环弹出队首，更新它4个邻居的max2start值，并压入队列。
初始化压入起始点，一直循环到队空为止。
这么做应该不会有错误，但可想而知，一个元素可能多次被压入队列，耗时太久，RunTimeError了2/3的TestCases……

下面说读满分代码发现的两种答案。
答案一：思路大致类似，但增加了等大Bool矩阵visited，而且是实用Priority Queue。
[\_LELOY\_的解法](https://www.hackerrank.com/rest/contests/hour-of-code-2016/challenges/the-great-robbery/hackers/_LELOY_/download_solution)
``` C++
#include <cmath>
#include <cstdio>
#include <vector>
#include <iostream>
#include <algorithm>
#include <queue>
#include <climits>
using namespace std;

typedef pair<int, int> ii;
typedef pair<int, ii>   iii;

class CompareDist
{
public:
    bool operator()(iii n1, iii n2) {
        return n1.first < n2.first;
    }
};

int main() {
    
    int N, M;       cin>>N>>M;
    vector<vector<int> > grid(N, vector<int>(M, 0));
    
    int sI, sJ;     cin>>sJ>>sI;
    int eI, eJ;     cin>>eJ>>eI;
    
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < M; ++j) {
            cin>>grid[i][j];
        }
    }
    
    vector<vector<int> > vis(N, vector<int>(M, false));
    
    priority_queue<iii, vector<iii> , CompareDist > q;
    q.push(make_pair(grid[sI][sJ], make_pair(sI, sJ)));
    
    while(!q.empty()) {
        int i = q.top().second.first;
        int j = q.top().second.second;
        int g = q.top().first;
        q.pop();
        //cout<<g<<" "<<i<<" "<<j<<" "<<vis[i][j];
        vis[i][j] = true;
        
        int dr[] = {0, -1, 0, 1};
        int dc[] = {-1, 0, 1, 0};
        
        for (int k = 0; k < 4; ++k) {
            int ti = i + dr[k];
            int tj = j + dc[k];
            if (ti < 0 || tj < 0 || ti >= N || tj >= M)
                continue;
            if (vis[ti][tj])
                continue;
            if (grid[i][j] < grid[ti][tj]) {
                grid[ti][tj] = grid[i][j];
                
            }
            q.push(make_pair(grid[ti][tj], make_pair(ti, tj)));
        }
        /*
        cout<<" done"<<endl;
        
        for (auto a: grid) {
            for (auto b: a) {
                cout<<b<<" ";
            }
            cout<<endl;
        }
        cout<<endl;
        */
    }
    
    cout<<grid[eI][eJ]<<endl;
    
    return 0;
}
```

答案二：二分查找的思想。这是我第一次见到。可以这么想：
既然需要找容量最大的可行路径，而且已知所有元素的值在[_Min_, _Max_]之间，那么解肯定也在这个范围内。那么我只要不断用一个中值med，测试是否有一条容量大于等于med的路径就行。
二分查找的精彩运用啊！
如何测试是否有这条路径呢？类似的，bool矩阵visited + 普通队列就可实现。

[cs1235的解法](https://www.hackerrank.com/rest/contests/hour-of-code-2016/challenges/the-great-robbery/hackers/cs1235/download_solution)
``` Java
import java.io.*;
import java.util.*;

public class Solution {

    static class Pair {
        int x;
        int y;
        public Pair(int x, int y) {
            this.x = x;
            this.y = y;
        }
    }
    
    public static boolean reachableWithWeight(int h, int w, int[][] grid, int startX, int startY, int destX, int destY, int weight) {
        if(weight > grid[startY][startX]) {
            return false;
        }
        boolean[][] visited = new boolean[h][w];
        Queue<Pair> queue = new LinkedList<Pair>();
        queue.add(new Pair(startX, startY));
        while(!queue.isEmpty()) {
            Pair next = queue.poll();
            int x = next.x;
            int y = next.y;
            if(x < 0 || x >= w || y < 0 || y >= h) {
                continue;
            }
            if(visited[y][x]) {
                continue;
            }
            //System.out.println(y + " " + x);
            visited[y][x] = true;
            if(weight > grid[y][x]) {
                continue;
            }
            if(x == destX && y == destY) {
                return true;
            }
            queue.add(new Pair(x + 1, y));
            queue.add(new Pair(x - 1, y));
            queue.add(new Pair(x, y + 1));
            queue.add(new Pair(x, y - 1));
        }
        return false;
    }
    
    public static void main(String[] args) throws IOException {
        BufferedReader f = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(f.readLine());
        int n = Integer.parseInt(st.nextToken());
        int m = Integer.parseInt(st.nextToken());
        st = new StringTokenizer(f.readLine());
        int startX = Integer.parseInt(st.nextToken());
        int startY = Integer.parseInt(st.nextToken());
        st = new StringTokenizer(f.readLine());
        int destX = Integer.parseInt(st.nextToken());
        int destY = Integer.parseInt(st.nextToken());
        int[][] grid = new int[n][m];
        for(int i = 0; i < n; i++) {
            st = new StringTokenizer(f.readLine());
            for(int j = 0; j < m; j++) {
                grid[i][j] = Integer.parseInt(st.nextToken());
            }
        }
        int lower = 1;
        int upper = 1000000000;
        while(lower < upper) {
            int med = (lower + upper)/2;
            //System.out.println(med);
            if(reachableWithWeight(n, m, grid, startX, startY, destX, destY, med)) {
                lower = med + 1;
            } else {
                upper = med;
            }
            //System.out.println();
            
        }
        System.out.println(lower - 1);
    }
}
```

最后放一个我参考方法一的Python代码，思路一样然而超时了…… 第一次用Python的PriorityQueue。
``` Python
import queue

n, m = input().strip().split(' ')
n, m = [int(n), int(m)]
sX, sY = input().strip().split(' ')
sX, sY = [int(sX), int(sY)]
dX, dY = input().strip().split(' ')
dX, dY = [int(dX), int(dY)]

ice = []
for i in range(n):
    a_t = [int(a_temp) for a_temp in input().strip().split(' ')]
    ice.append(a_t)   
#print(ice)

visited = [[False for col in range(m)] for row in range(n)]

q = queue.PriorityQueue()
q.put((-ice[sY][sX], [sY, sX]))
while(not q.empty()):
    y, x = q.get()[1]
    
    if (visited[y][x]):
        continue
    if (dY == y and dX == x):
        break
        
    visited[y][x] = True
    #print('update [',y,'][',x,']')
    
    for [ny, nx] in [[y-1,x], [y,x-1], [y,x+1], [y+1,x]]:
        if (ny < 0 or ny >= n or nx < 0 or nx >= m):
            continue
        if (visited[ny][nx]):
            continue

        if (ice[y][x] < ice[ny][nx]):
            ice[ny][nx] = ice[y][x]
        q.put((-ice[ny][nx], [ny, nx]))
        #print('= append [',ny,'][',nx,'], new Value =', ice[ny][nx])

print(ice[dY][dX])
```