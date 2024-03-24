[toc]

[leetcode 第 390 场周赛](https://leetcode.cn/contest/weekly-contest-390/)

# [100245. 每个字符最多出现两次的最长子字符串](https://leetcode.cn/problems/maximum-length-substring-with-two-occurrences/)

**题意**

给定一个长度小于等于 100 的仅由小写字母构成的字符串，请你找一个最长的子串，满足其中每种字符最多出现两次

**思路**

考虑到数据范围，可以直接**暴力**判断所有可能的字串是否满足题意，记录过程中的最大值即可。

时间复杂度 $O(n^2)$



看到本题的第一眼就是**双指针**的思路。定义一前一后两个指针，前指针一直向前，统计途中的字符出现次数，直到出现次数大于 2 的字符为止，此时后指针往前跑直至该字符的出现次数小于等于 2 为止；前指针再次出发，重复上述行为；过程中记录下满足题意的最大值即可。

时间复杂度 $O(n)$



**代码**

```cpp
class Solution {
public:
    int maximumLengthSubstring(string s) {
        int ans = 1;
        int l = 0, r = 0, n = s.size();
        vector<int> cnt(26);
        while(r < n){
            ++ cnt[s[r] - 'a'];
            if(cnt[s[r] - 'a'] > 2){
                while(l < r && cnt[s[r] - 'a'] > 2){
                    -- cnt[s[l] - 'a'];
                    ++ l;
                }
            }else{
                ans = max(ans, r - l + 1);
            }
            ++ r;
        }
        return ans;
    }
};
```



# [100228. 执行操作使数据元素之和大于等于 K](https://leetcode.cn/problems/apply-operations-to-make-sum-of-array-greater-than-or-equal-to-k/)

**题意**

给定一个整数 k，你拥有一个初始数组，里面只有数字 1。

你可以执行两种操作：

- 选择一个数组里面的元素，使其 + 1

- 选择一个数组里面的元素，在当前数组中再添加一个该元素

问最终数组元素之和大于等于 k 所需的最少操作次数？



**思路**

可以考虑到，如果说最终的操作需要若干次 + 1 操作和复制操作，那么 + 1 操作一定出现在复制操作之前

所以我们优先 + 1 操作，再利用其进行复制操作。

考虑到 $1 \le k \le 10^5$，所以可以枚举 + 1 的次数，过程中取最小的总操作次数即可



**代码**

```cpp
class Solution {
public:
    int minOperations(int k) {
        const int inf = 0x3f3f3f3f;
        int ans = inf;
        for(int i = 1; i <= k; ++ i){
            ans = min(ans, (i - 1) + (k + i - 1) / i - 1);
        }
        return ans;
    }
};
```



# [100258. 最高频率的 ID](https://leetcode.cn/problems/most-frequent-ids/description/)

**题意**

给定两个长度均为 `n` 的数组 `nums` 和 `freq`，`freq` 数组中的数表示：初始集合为空，对于数组 `freq` 中的第 `i` 个数 `freq[i]`，将会在该集合中添加 `freq[i]` 个数 `nums[i]`（`freq[i] < 0` 时代表删除数）

要求你返回一个长度为 `n` 的数组，标识每一次操作后，当前集合中出现频率最高的元素的数目

$1 \le n 、nums[i] \le 10^5，-10^5 \le freq[i] \le 10^5$



**思路**

看到 1e5 范围的统计频率，联想到单点修改，区间查询最大值的**线段树**



也可以利用 **multiset** 来动态维护频率最大值



**代码**

- 线段树

```cpp
#define ll long long
struct Segment_Tree{//update - range_add + query - range_max
	int n;
	vector<ll> seg, tag, val;
	Segment_Tree(int cnt) : n(cnt), seg(cnt << 2, 0), tag(cnt << 2, 0), val(cnt + 1, 0){}
	int ls(int p) { return p << 1; }
	int rs(int p) { return p << 1 | 1; }

	void push_up(int p){ seg[p] = max(seg[ls(p)], seg[rs(p)]); return ;	}
	void build(int p, int pl, int pr){
		tag[p] = 0;
		if(pl == pr){
			seg[p] = val[pl]; return ;
		}
		int mid = pl + pr >> 1;
		build(ls(p), pl, mid);
		build(rs(p), mid + 1, pr);
		push_up(p);
		return ;
	}

	void addtag(int p, int pl, int pr, ll k){
		tag[p] += k;
		seg[p] += (pr - pl + 1) * k;
		return ;
	}

	void push_down(int p, int pl, int pr){
		if(tag[p]){
			int mid = pl + pr >> 1;
			addtag(ls(p), pl, mid, tag[p]);
			addtag(rs(p), mid + 1, pr, tag[p]);
			tag[p] = 0;
		}
		return ;
	}

	void update(int p, int pl, int pr, int l, int r, ll k){
		if(l <= pl && pr <= r){
			addtag(p, pl, pr, k);
			return ;
		}
		push_down(p, pl, pr);
		int mid = pl + pr >> 1;
		if(l <= mid) update(ls(p), pl, mid, l, r, k);
		if(mid < r) update(rs(p), mid + 1, pr, l, r, k);
		push_up(p);
		return ;
	}

	ll query(int p, int pl, int pr, int l, int r){
		if(l <= pl && pr <= r){
			return seg[p];
		}
		push_down(p, pl, pr);
		ll res = 0;
		int mid = pl + pr >> 1;
		if(l <= mid) res = max(res, query(ls(p), pl, mid, l, r));
		if(mid < r) res = max(res, query(rs(p), mid + 1, pr, l, r));
		return res;
	}
};

class Solution {
public:
    vector<long long> mostFrequentIDs(vector<int>& a, vector<int>& q) {
        int n = q.size(), mx = 1e5;
        vector<long long> ans(n);
        Segment_Tree tree(mx);
        for(int i = 0; i < n; ++ i){
            tree.update(1, 1, mx, a[i], a[i], q[i]);
            ans[i] = tree.query(1, 1, mx, 1, mx);
        }
        return ans;
    }
};
```

- multiset

```cpp
class Solution {
public:
    vector<long long> mostFrequentIDs(vector<int>& a, vector<int>& q) {
        int n = a.size();
        vector<long long> ans(n);
        multiset<long long> f;
        map<int, long long> cnt;
        for(int i = 0; i < n; ++ i){
            if(cnt[a[i]])
                f.erase(f.find(cnt[a[i]]));
            cnt[a[i]] += q[i];
            f.insert(cnt[a[i]]);
            if(f.size()) ans[i] = *f.rbegin();
        }
        return ans;
    }
};
```



# [100268. 最长公共后缀查询](https://leetcode.cn/problems/longest-common-suffix-queries/description/)

**题意**

给定一组字符串和一组字符串询问，对于每一个询问的字符串，在给定的字符串中找到一个与询问字符串有最长公共后缀的字符串，如果有多个满足题意得则优先找长度短的、下标小的

给定字符串的总长度和询问字符串的总长度均小于 5e5，字符串个数均小于 1e4

**思路**

**Trie树插入和查询**

对于树上的每一个节点都打上两个标记：id 和 len，id 记录当前节点标记的后缀所对应的匹配的字符串的下标，len 即为该字符串的长度。如此，在插入字符串时维护这两个标记即可



**代码**

```cpp
struct Trie{
    // using ll = long long;
    enum {alpha = 26, st = 'a'};
    struct node{
        int id = 0, len = INT_MAX;// 保存id
        int ch[alpha] = {0};
        node(){}
    };
    vector<node> tr;
    int new_node(){
        tr.emplace_back();
        return tr.size() - 1;
    }
    Trie(){ new_node(); }

    void judge(int x, int y, int size){
        if(size < tr[x].len || size == tr[x].len && y < tr[x].id){
            tr[x].len = size;
            tr[x].id = y;
        }
        return ;
    }

    void insert(const string & ss, int id, int size){
        int p = 0;// 根节点
        for(int i = ss.size() - 1; i >= 0; -- i){
            char c = ss[i];
            if(!tr[p].ch[c - st])
                tr[p].ch[c - st] = new_node();
            judge(p, id, size);
            p = tr[p].ch[c - st];
        }
        judge(p, id, size);
        return ;
    }

    int find(const string & ss){
        int p = 0;
        for(int i = ss.size() - 1; i >= 0; -- i){
            char c = ss[i];
            if(!tr[p].ch[c - st]) return tr[p].id;
            p = tr[p].ch[c - st];
        }
        return tr[p].id;
    }
};

class Solution {
public:
    vector<int> stringIndices(vector<string>& ss, vector<string>& q) {
        Trie tree;
        for(int i = 0; i < ss.size(); ++ i){
            tree.insert(ss[i], i, ss[i].size());
        }
        vector<int> ans;
        for(auto& s : q){
            ans.push_back(tree.find(s));
        }
        return ans;
    }
};
```

