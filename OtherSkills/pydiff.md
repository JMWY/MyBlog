# python  多进程并发 diff 脚本

## A 
```python
#!/bin/env /usr/bin/python2.7
# coding=utf-8
import sys
import os
import signal
import urllib2
import xml.dom.minidom as xmldom
import time
import math
#from multiprocessing import Pool, cpu_count, current_process
from multiprocessing import Pool
import pdb

server_addr_base = "http://11.12.195.168:8828/"
#server_addr_new = "http://11.12.195.168:8828/"
server_addr_new = "http://11.138.12.90:8828/"

def first_elem(elem):
    return elem[0]

def get_poilist(result):
        res = []
    try:
            xmlobj = xmldom.parseString(result)
            eobj = xmlobj.documentElement
            subobj = eobj.getElementsByTagName("hits")[0].getElementsByTagName("hit")
            for id in range(len(subobj)):
                    res.append(subobj[id].getElementsByTagName("pguid")[0].firstChild.data.encode('utf-8'))
    except Exception as err:
        print "parse response failed!"
        return ["parse_error"]
    return res

def do_requst(query):
    # pdb.set_trace()
    #print query
    try:
        resp = urllib2.urlopen(query)
    except Exception as err:
        try:
            resp = urllib2.urlopen(query)
        except Exception as err:
            print "request ha3 server failed!"
            return []
        return get_poilist(resp.read())

def has_diff(listA, listB):
    # print "A:", listA
    # print "B:", listB

    # compare ordered
    return listA != listB
    
    # compare unordered
    # return set(listA) != set(listB)

def do_diff_core(sid, eid, lines):
    diff_cnt = 0
    diff_req = []
    for reqid in range(sid, eid):
        try:
            line = lines[reqid].strip()
            req = urllib2.quote(line)
        except Exception as err:
            print "[Fatal] line:", line
            sys.exit(0)

        A = do_requst(server_addr_base + req)
        B = do_requst(server_addr_new + req)
        if has_diff(A, B):
            diff_cnt += 1
            diff_req.append(line)
    return (sid, diff_cnt, diff_req)

def sig_handle(sig, frame):
    print "ignore Ctrl+C ..."

def dispatch(reqlist):
    parallel_num = 20 # cpu_count()
    signal.signal(signal.SIGTERM, sig_handle)
    signal.signal(signal.SIGINT, sig_handle)
    pool = Pool(parallel_num)
    ret_set = []
    result_set = []
    if len(reqlist) >= parallel_num:
        step = int(math.ceil(len(reqlist)/parallel_num))
        sid = 0
        while sid < len(reqlist):
            eid = sid + step
            if eid > len(reqlist):
                eid = len(reqlist)
            print "dispatch request segment:[%d, %d)" % (sid, eid)
            ret_set.append(pool.apply_async(do_diff_core, (sid, eid, reqlist, )))
            sid = eid
        result_set = [x.get() for x in ret_set]
    else:
        result_set.append(do_diff_core(0, len(reqlist), reqlist))
    result_set.sort(key=first_elem, reverse=False)

    pool.close()
    pool.join()
    
    diff_num = 0
    diff_reqlist = []
    for r in result_set:
        diff_num += r[1]
        if len(r[2]) > 0:
            diff_reqlist.extend(r[2])
    return diff_num, diff_reqlist

def do_diff(reqfile, outfile=None):
    stime = time.time()
    if outfile is None:
        outfile = reqfile + ".diff.out"
    fd = open(reqfile, 'r')
    os.system("[ -f %s ] && mv %s %s.bak " % (outfile, outfile, outfile))
    fd2 = open(outfile, 'w')
    req_cnt = 0
    diff_cnt = 0
        lines = fd.readlines(10000000)
    while lines:
        diff_num, diff_reqlist = dispatch(lines)
        diff_cnt += diff_num
        for diff_req in diff_reqlist:
            fd2.write(diff_req + '\n')
        
        req_cnt += len(lines)
        print req_cnt, " done..."
            lines = fd.readlines(10000000)
    fd.close()
    fd2.close()
    etime = time.time()
    print "elapsed time is ", (etime-stime), " s"
    print "diff number is:", diff_cnt
    #if diff_cnt == req_cnt:
    #    sys.exit(0)
    return diff_cnt

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print "no Usage"
             exit(0)
    input_file = reqfile = sys.argv[1]
    output_file = outfile = reqfile + ".diff.out"
    os.system("sed -i 's/format:protobuf/format:xml/g' %s" % reqfile)
    diff_num = do_diff(reqfile, outfile)
    if diff_num == 0:
        sys.exit(0)
    retry = 0
    while diff_num > 0:
        retry += 1
        print "retry ", retry, " times..."
        reqfile = "retry_" + str(retry)
        if reqfile == sys.argv[1]:
            reqfile = str(retry)
        os.system("cp %s %s" % (outfile, reqfile))
        outfile = reqfile  + ".diff.out"
        diff_num_new = do_diff(reqfile, outfile)
        if diff_num_new == diff_num:
            break
        diff_num = diff_num_new
    os.system("cp %s %s && rm -f retry_*" %(outfile, output_file))
    sys.exit(0)

```

## B

```python
# author: liyangguang
#!/bin/env /usr/bin/python2.7
# coding=utf-8
import sys
import os
import urllib2
import xml.dom.minidom as xmldom
import urllib
from lxml import etree
import json
import time
import math
from multiprocessing import Pool, cpu_count, current_process
import pdb

server_addr_base = "http://11.12.195.168:3098/"
server_addr_new = "http://11.138.12.90:3098/"

def first_elem(elem):
    return elem[0]
        
def is_slayer(req):
    return req.find("query_type=slayer") > 0

def get_poilist(result, is_slayer):
        res = []
    if is_slayer:
        return res
    try:
        root=etree.fromstring(result)
        nodes=root.xpath('/LbsResult/KeywordRes/list/searchresult/list/poi/pguid')
    except Exception as err:
        print "parse response failed!"
        return ["parse_error"]
        for id in range(len(nodes)):
            res.append(nodes[id].text)
    return res

def do_requst(query):
    # pdb.set_trace()
    # print query
    try:
        resp = urllib.urlopen(query)
    except Exception as err:
        try:
            resp = urllib.urlopen(query)
        except Exception as err:
            print "request sp server failed!"
            return []
        return get_poilist(resp.read(), is_slayer(query))

def has_diff(listA, listB):
    #print "A:", listA
    #print "B:", listB
    if len(listA) != len(listB):
        return True

    for id in range(len(listA)):
        if listA[id] != listB[id]:
            return True
    return False

def do_diff_core(sid, eid, lines):
    diff_cnt = 0
    diff_req = []
    for reqid in range(sid, eid):
        try:
            req = lines[reqid].strip()
        except Exception as err:
            print "[Fatal] line:", line
            sys.exit(0)

        A = do_requst(server_addr_base + req)
        B = do_requst(server_addr_new + req)
        if has_diff(A, B):
            diff_cnt += 1
            diff_req.append(req)
    return (sid, diff_cnt, diff_req)


def dispatch(reqlist):
    parallel_num = 20 # cpu_count()
    pool = Pool(parallel_num)
    ret_set = []
    result_set = []
    if len(reqlist) >= parallel_num:
        step = int(math.ceil(len(reqlist)/parallel_num))
        sid = 0
        while sid < len(reqlist):
            eid = sid + step
            if eid > len(reqlist):
                eid = len(reqlist)
            print "dispatch request segment:[%d, %d)" % (sid, eid)
            ret_set.append(pool.apply_async(do_diff_core, (sid, eid, reqlist, )))
            sid = eid
        result_set = [x.get() for x in ret_set]
    else:
        result_set.append(do_diff_core(0, len(reqlist), reqlist))
    result_set.sort(key=first_elem, reverse=False)

    pool.close()
    pool.join()
    
    diff_num = 0
    diff_reqlist = []
    for r in result_set:
        diff_num += r[1]
        if len(r[2]) > 0:
            diff_reqlist.extend(r[2])
    return diff_num, diff_reqlist

def do_diff(reqfile, outfile=None):
    stime = time.time()
    if outfile is None:
        outfile = reqfile + ".diff.out"
    fd = open(reqfile, 'r')
    os.system("[ -f %s ] && mv %s %s.bak " % (outfile, outfile, outfile))
    fd2 = open(outfile, 'w')
    req_cnt = 0
    diff_cnt = 0
        lines = fd.readlines(10000000)
    while lines:
        diff_num, diff_reqlist = dispatch(lines)
        diff_cnt += diff_num
        for diff_req in diff_reqlist:
            fd2.write(diff_req + '\n')
        
        req_cnt += len(lines)
        print req_cnt, " done..."
            lines = fd.readlines(10000000)
    fd.close()
    fd2.close()
    etime = time.time()
    print "elapsed time is ", (etime-stime), " s"
    print "diff number is:", diff_cnt
    #if diff_cnt == req_cnt:
    #    sys.exit(0)
    return diff_cnt

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print "no Usage"
             exit(0)
    input_file = reqfile = sys.argv[1]
    output_file = outfile = reqfile + ".diff.out"
    diff_num = do_diff(reqfile, outfile)
    retry = 0
    while diff_num > 0:
        retry += 1
        print "retry ", retry, " times..."
        reqfile = "retry_" + str(retry)
        if reqfile == sys.argv[1]:
            reqfile = str(retry)
        os.system("cp %s %s" % (outfile, reqfile))
        outfile = reqfile  + ".diff.out"
        diff_num_new = do_diff(reqfile, outfile)
        if diff_num_new == diff_num:
            break
        diff_num = diff_num_new
    os.system("cp %s %s && rm -f retry_*" %(outfile, output_file))

```
