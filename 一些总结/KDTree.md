

在讲KDTree之前首先要先讲一下树。

树是一种索引结构，可以按照一定次序存储数据节点。比如二叉搜索树，比当前节点key 值大的节点存储在节点的右子树，比当前节点key 值小的节点存储在节点的左子树。

树的存储结构有什么好处呢？它可以缩短我们的查找时间（相比于链表），对于链表来说，查找key 为某一值的节点的时间复杂度是O(n)，而对于二叉搜索树来说，虽然时间复杂度也是O(n)，但是它的整体速度会快一些。而对于平衡二叉树来说，查找的时间复杂度为O(log n)。对于插入操作也是类似的。

除了二叉树，二叉搜索树，平衡二叉树，树的变体还有很多，比如红黑树（这个我不太了解），B树，B+树，2-3-4树，KDTree等等。其中，B树，2-3-4树上学期编程实验写过，现在已经忘的差不多了，什么时候需要再回顾一下。

这篇博客主要讲的就是KDTree，KDTree就是K-dimentional Tree的缩写，也就是K维树，它的key是k维数据，而我们前面所说二叉树的key是一维的。一个典型的例子就是二维数据，坐标。坐标之间是可以比较距离的，根据坐标之间的方位，一个直观的想法是我们可以构建一个四叉树，分别代表左上，左下，右上，右下。我们也可以构建一个KDTree，每层节点判定的标准不同，如果当前层判定的是x坐标大小，则下一层判定y坐标。

如果我们要查找树中距离某一点t距离最近的节点，最朴素的想法就是遍历KDTree的所有节点，记录一个最近节点。这与二叉树不同，二叉树只需遍历左子树或者右子树，而对于KDTree来说，没有这个性质，因为它的判断标准是二维的，但是我们可以剪枝，对于当前节点，我们可以记录一个goodSide和badSide。比如当前节点的判别依据是x，如果`t.x < curr.x`, 则goodSide就是左子树，badSide就是右子树。goodSide无论如何我们都是要去遍历的，但是badSide我们可以先判断一下，如果badSide中到t最近的点都大于当前记录的最近节点到t的距离，则badSide就不用考虑了。

下面用c实现了一个KDTree，背景是澳洲某个地方店铺的数据，每条数据中包含了：Block ID, x, y, 店铺名称等等，需要注意有坐标重复的店铺。有两个任务，任务一是给的某个位置的x,y坐标，找到距离当前位置的最近的店铺，并输出他们的所有信息。任务二是给定某个位置的x,y坐标和半径r，找出距离当前位置在r内的所有店铺，并输出。

这里作为key的是x和y坐标，因为可能会有坐标重复的店铺，所以KDTree节点中包含一个链表，存储坐标相同的店铺信息。

首先就要读数据，这里数据是一个csv文件，c读写文件这块比较麻烦，一定需要特别注意。将文件中的信息读取，建立一个KDTree，然后搜索。

```C
//main.c
/*
    A map1 based on the CLUE database
    Run with
        map1 datafile outputfile < inputfile
    
    where 
        datafile is a CLUE csv,
        outputfile is the name of the file which data will be output to,
    
    and
        inputfile is in the format of the business name to search for,
        with one business name per line.
    written by lovelyfrog
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>
#include "read.h"
#include "map.h"
#include "utils.h"

int main(int argc, char **argv){
    if(argc < 3){
        fprintf(stderr, "Incorrect arguments\n");
        fprintf(stderr, "Run in the form\n");
        fprintf(stderr, "\tmap1 datafile outputfile < inputfile\n");
        exit(1);
    }
    
    char *line = NULL;
    size_t size = 0;
    
    struct map *map = readFile(argv[1]);
    struct searchResult *result = NULL;

    
    FILE *outputFile = fopen(argv[2], "w");
    assert(outputFile);
    
    while(getlineClean(&line, &size, stdin) != (-1)){
        result = queryMap1(map, line);
        writeMap1SearchResult(result, map, outputFile);
        freeSearchResult(result);
    }
    
    if(line){
        free(line);
    }
    if(map){
        freeMap(map);
    }
    if(outputFile){
        fclose(outputFile);
    }
    
    return 0;
}

```

其中`map`是这样的：

```c
struct map {
  	// mapping中存储了所有的header, key的location, data的location等
    struct dataMapping *mapping;
    struct treeNode *root;
};

struct dataMapping {
    char **headers;
    int headerCount;
    /* Maps each key field to its index in the row. */
    int *keyLocations;
    /* Maps each data field to its index in the row. */
    int *dataLocations;
    int keyCount;
};
```

建树：

```C
// read.c
/*
    This defines implementations for read-related functions.
*/

#include "map.h"
#include <stdlib.h>
#include <stdio.h>
#include <assert.h>
#include "utils.h"

struct map *readFile(char *filename){
    FILE *file = NULL;
    struct map *returnMap = NULL;
    file = fopen(filename, "r");
    assert(file);
    
    char *line = NULL;
    size_t size = 0;
    
    /* First line is headers, so we handle differently. */
    if(getlineClean(&line, &size, file) != (-1)){
        returnMap = newMap(line);
    }
    
    /* Rest of lines are rows. */
    while(getlineClean(&line, &size, file) != (-1)){
        prependRow(returnMap, line);
    }
    
    if(line){
        free(line);
    }
    if(file){
        fclose(file);
    }
    return returnMap;
}

```

treeNode是这样定义的：

```C
struct treeNode {
    int checkX;
    double x;
    double y;
    struct treeNode *left;
    struct treeNode *right;
    struct ll *next;
};

struct data {
    char **data;
};

struct key {
    char **keys;
};
```

```C
//map.c
/*
    This defines implementations for map1-related functions.
*/

#include "map.h"
#include "data.h"
#include "ll.h"
#include "kdtree.h"
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <string.h>

#define KEYDATAJOIN "-->"
#define FAILTEXT "NOTFOUND"

struct map *newMap(char *header){
    struct map *returnMap = (struct map *) malloc(sizeof(struct map));
    assert(returnMap);
    returnMap->mapping = getDataMapping(header);
    returnMap->root = NULL;
    return returnMap;
}

void prependRow(struct map *map, char *row){
    prependToKDTree(&map->root, map->mapping, row);
}

struct searchResult *queryMap1(struct map *map, char *string){
    struct key *searchKey = readKey(string, 2);
    struct searchResult *result = (struct searchResult *) 
        malloc(sizeof(struct searchResult));
    assert(result);
    result->key = searchKey;
    result->data = NULL;
    result->mapping = map->mapping;
    result->keyComparedNum = 0;
    double x = string2double((searchKey->keys)[0]);
    double y = string2double((searchKey->keys)[1]);
    searchNearest(map->root, map->mapping, x, y, &(result->data), &(result->keyComparedNum));
    
    return result;
}

struct searchResult *queryMap2(struct map *map, char *string) {
    struct key *searchKey = readKey(string, 3);
    struct searchResult *result = (struct searchResult *) 
        malloc(sizeof(struct searchResult));
    assert(result);
    result->key = searchKey;
    result->data = NULL;
    result->mapping = map->mapping;
    result->keyComparedNum = 0;
    double x = string2double((searchKey->keys)[0]);
    double y = string2double((searchKey->keys)[1]);
    double r = string2double((searchKey->keys)[2]);
    searchWithinRadius(map->root, map->mapping, x, y, r, &(result->data), &(result->keyComparedNum));
    return result;
}

void writeMap1SearchResult(struct searchResult *result, struct map *map, FILE *file){
    char *keyText;
    char *resultText;
    int i;
    int first = 1;
    
    keyText = getKeyString(map->mapping, result->key);
    
    i = 0;
    while((result->data)[i]){
        fprintf(file, "%s", keyText);
        resultText = getDataString(map->mapping, (result->data)[i]);
        fprintf(file, " %s %s\n", KEYDATAJOIN, resultText);
        if(resultText){
            free(resultText);
        }
        i++;
        first = 0;
    }
    if(first){
        fprintf(file, "%s\n", FAILTEXT);
    }
    if(keyText){
        free(keyText);
    }
    fprintf(stdout, "%s %s ", keyText, KEYDATAJOIN);
    fprintf(stdout, "%d\n", result->keyComparedNum);
}

void writeMap2SearchResult(struct searchResult *result, struct map *map, FILE *file, char *string) {
    struct key *searchKey = readKey(string, 3);
    char *keyText;
    char *resultText;
    int i;
    int first = 1;
    
    keyText = getKeyString(map->mapping, result->key);
    
    i = 0;
    while((result->data)[i]){
        fprintf(file, "%s ", keyText);
        fprintf(file, "%s", searchKey->keys[2]);
        resultText = getDataString(map->mapping, (result->data)[i]);
        fprintf(file, " %s %s\n", KEYDATAJOIN, resultText);
        if(resultText){
            free(resultText);
        }
        i++;
        first = 0;
    }
    if(first){
        fprintf(file, "%s\n", FAILTEXT);
    }
    if(keyText){
        free(keyText);
    }
    fprintf(stdout, "%s %s ", keyText, KEYDATAJOIN);
    fprintf(stdout, "%d\n", result->keyComparedNum);
}

void freeMap(struct map *map){
    freeKDTree(map->root, map->mapping);
    freeDataMapping(&(map->mapping));
    free(map);
}

void freeSearchResult(struct searchResult *result){
    if(result){
        freeKey(&(result->key), result->mapping);
        if(result->data){
            free(result->data);
        }
        free(result);
    }
}

```

```C
//kdtree.c
/*
    This defines implementations for KD-tree related functions
*/

#include "data.h"
#include "ll.h"
#include "kdtree.h"
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <math.h>


struct treeNode *createKDTreeNode(int checkX, double x, double y, struct treeNode *left, struct treeNode *right, struct ll *next) {
    struct treeNode *returnTreeNode = (struct treeNode *) malloc(sizeof(struct treeNode));
    returnTreeNode->checkX = checkX;
    returnTreeNode->x = x;
    returnTreeNode->y = y;
    returnTreeNode->left = left;
    returnTreeNode->right = right;
    returnTreeNode->next = next;
    return returnTreeNode; 
}

void getXY(struct key *key, double *x, double *y) {
    char **locations = key->keys;
    char *xCoordinate = locations[0];
    char *yCoordinate = locations[1];
    *x = string2double(xCoordinate);
    *y = string2double(yCoordinate);
}

double string2double(char *str) {
    return strtod(str, NULL);
}

void prependToKDTree(struct treeNode **root, struct dataMapping *mapping, char *row) {
    struct ll *newHead = newNode();
    getData(row, mapping, &(newHead->key), &(newHead->data));
    struct treeNode **currentTreeNode = root;
    int checkX = 1;
    prependNodeToKDTree(currentTreeNode, mapping, newHead, checkX);
}

void prependNodeToKDTree(struct treeNode **node, struct dataMapping *mapping, struct ll *newHead, int checkX) {
    double targetX, targetY;
    getXY(newHead->key, &targetX, &targetY);
    if (*node == NULL) {
        *node = createKDTreeNode(checkX, targetX, targetY, NULL, NULL, newHead);
        return;
    }
    double nodeCheck, nodeNotCheck, targetCheck, targetNotCheck;
    if (checkX == 1) {
        nodeCheck = (*node)->x;
        nodeNotCheck = (*node)->y;
        targetCheck = targetX;
        targetNotCheck = targetY;
    } else {
        nodeCheck = (*node)->y;
        nodeNotCheck = (*node)->x;
        targetCheck = targetY;
        targetNotCheck = targetX;        
    }
    if (targetCheck < nodeCheck) {
        prependNodeToKDTree(&(*node)->left, mapping, newHead, 1-checkX);
    } else {
        if (targetCheck == nodeCheck && targetNotCheck == nodeNotCheck) {
            newHead->next = (*node)->next;
            (*node)->next = newHead;
        } else {
            prependNodeToKDTree(&(*node)->right, mapping, newHead, 1-checkX);
        }
    }
}

double getDistance(double x1, double y1, double x2, double y2) {
    return sqrt(pow(x1 - x2, 2) + pow(y1 - y2, 2));
}

void searchBestTreeNode(struct treeNode *curr, struct treeNode **best, double x, double y, int *keyComparedNum) {
    if (curr == NULL) {return;}
    if (getDistance(curr->x, curr->y, x, y) < getDistance((*best)->x, (*best)->y, x, y) ) {
        *best = curr;
    }
    (*keyComparedNum)++;
    struct treeNode *goodSide = NULL;
    struct treeNode *badSide = NULL;
    if (curr->checkX == 1) {
        if (x < curr->x) {
            goodSide = curr->left;
            badSide = curr->right;
        } else {
            goodSide = curr->right;
            badSide = curr->left;
        }
    } else {
        if (y < curr->y) {
            goodSide = curr->left;
            badSide = curr->right;
        } else {
            goodSide = curr->right;
            badSide = curr->left;
        }
    }
    (*keyComparedNum)++;
    searchBestTreeNode(goodSide, best, x, y, keyComparedNum);
    if ((curr->checkX == 1 && getDistance(x, 0, curr->x, 0) <= getDistance((*best)->x, (*best)->y, x, y) ) || (curr->checkX == 0 &&  getDistance(0, y, 0, curr->y) <= getDistance((*best)->x, (*best)->y, x, y))) {
        searchBestTreeNode(badSide, best, x, y, keyComparedNum);
    }
    (*keyComparedNum)++;
    return;
}

void searchWithinRadiusTreeNode(struct treeNode *curr, struct treeNode ***withinRadius, double x, double y, double r, int *keyComparedNum, int *num) {
    if (curr == NULL) {return;};
    if (getDistance(curr->x, curr->y, x, y) <= r) {
        *withinRadius = realloc(*withinRadius, sizeof(struct treeNode *) * (*num + 2));
        (*withinRadius)[*num] = curr;
        (*withinRadius)[(*num)+1] = NULL;
        (*num)++;
    }
    (*keyComparedNum)++;
    struct treeNode *goodSide = NULL;
    struct treeNode *badSide = NULL;
    if (curr->checkX == 1) {
        if (x < curr->x) {
            goodSide = curr->left;
            badSide = curr->right;
        } else {
            goodSide = curr->right;
            badSide = curr->left;
        }
    } else {
        if (y < curr->y) {
            goodSide = curr->left;
            badSide = curr->right;
        } else {
            goodSide = curr->right;
            badSide = curr->left;
        }
    }
    (*keyComparedNum)++;
    searchWithinRadiusTreeNode(goodSide, withinRadius, x, y, r, keyComparedNum, num);
    if ((curr->checkX == 1 && getDistance(x, 0, curr->x, 0) <= r ) || (curr->checkX == 0 &&  getDistance(0, y, 0, curr->y) <= r) ) {
        searchWithinRadiusTreeNode(badSide, withinRadius, x, y, r, keyComparedNum, num);
        (*keyComparedNum)++;
    }
}
// 注意这里面为什么放的是 ***data, 我们需要改变这个*data的值。如果放**data的话，就会有问题
void searchNearest(struct treeNode *root, struct dataMapping *mapping, double x, double y, struct data ***data, int *keyComparedNum) {
  
    struct treeNode *best = root;
    searchBestTreeNode(root, &best, x, y, keyComparedNum);
    struct ll *current = best->next;
    int foundMatches = 0;
    *data = (struct data **) malloc(sizeof(struct data *));
    **data = NULL;
    while (current != NULL) {
        *data = realloc(*data, sizeof(struct data *) * (foundMatches + 2));
        assert(*data);
        (*data)[foundMatches] = current->data;
        (*data)[foundMatches + 1] = NULL;
        foundMatches++;
        current = current->next;
    }
}

void searchWithinRadius(struct treeNode *root, struct dataMapping *mapping, double x, double y, double r, struct data ***data, int *keyComparedNum) {
    struct treeNode **withinRadius = NULL;
    int num = 0;
    searchWithinRadiusTreeNode(root, &withinRadius, x, y, r, keyComparedNum, &num);
    *data = (struct data **) malloc(sizeof(struct data *));
    **data = NULL;
    int foundMatches = 0;
    for (int i = 0; i < num; i++) {
        struct ll *current = withinRadius[i]->next;
        while (current != NULL) {
            *data = realloc(*data, sizeof(struct data *) * (foundMatches + 2));
            assert(*data);
            (*data)[foundMatches] = current->data;
            (*data)[foundMatches + 1] = NULL;
            foundMatches++;
            current = current->next;
        }
    }
}

void freeKDTree(struct treeNode *root, struct dataMapping *mapping) {
    if (!root) {
        return;
    }
    struct ll *current = root->next;
    freeLL(current, mapping);
    freeKDTree(root->left, mapping);
    freeKDTree(root->right, mapping);
    free(root);
}
```



