#!/usr/bin/env python3 
#
#  The MIT License (MIT)
#
# Copyright 2018 University of North Carolina at Chapel Hill. All other rights reserved.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy of this software
# and associated documentation files (the "Software"), to deal in the Software without
# restriction, including without limitation the rights to use, copy, modify, merge, publish,
# distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all copies or
# substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
# FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
# COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN
# AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
# WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#

import subprocess
import json
import os
import sys
import signal

TIMEOUT=10
# top-level subroutine or function takes no arguments
# code for OpenC2 consumer
def main():
   # handle HTTP requests within 10s for UNIX/LINUX
   signal.signal(signal.SIGALRM, handler)
   signal.alarm(TIMEOUT)  
   # call function that will handle incoming requests from an OpenC2 producer
   request_handler()
   signal.alarm(0)

# alarm function to timeout requests
def handler(signum, frame):
   server_error(); 
   raise IOError("Couldn't open device!")

#function to handle GET, POST, cli request and make calls to other functions to execute commands
#figure out exactly what we want this function to do as far as parsing
def request_handler():
   conformance=0
   openc2_actions=['query', 'deny', 'allow', 'update', 'delete']
   openc2_targets=['file', 'ipv4_net', 'ipv6_net', 'ipv4_connection', 'ipv6_connection', 'features', 'slpf:rulenumber']
   openc2_query=['versions','profiles','pairs','rate_limit']
   openc2_response_requested=['none','ack','status','complete']
   complete_response=True
   openc2_query_versions=['1.0']
   request=sys.stdin.read()
   parsed_json = json.loads(request)
   #do we have an actions var
   for alpha in openc2_actions:
      if alpha==parsed_json['action']:
         conformance += 1
   if conformance!=1:
      not_parseable()
   else: 
      conformance=0

   #was no response requested
   for epsilon in openc2_response_requested:
      try:
         resp_req=parsed_json.get("args").get("response_requested")
         if resp_req==openc2_response_requested[0]:
            sys.exit()
         elif resp_req==openc2_response_requested[1]:
            complete_response=False
         elif resp_req==openc2_response_requested[2]:
            complete_response=False
         elif resp_req==openc2_response_requested[3]:
            complete_response=True
         else:
            not_parseable()
      except:
         complete_response=True

   #do we have a targets var
   for beta in openc2_targets:
      if beta in parsed_json.get("target"):
         conformance += 1
   if conformance!=1:
      not_parseable()
   else:
      # we have a request with an action and a target
      conformance=0
      # check for query features
      feature_count=0
      feature_power=0
      feature_total=0
      if parsed_json.get("action")==openc2_actions[0]:
         if openc2_targets[5] in parsed_json.get("target"):
            for charlie in parsed_json.get("target").get("features"):
               if charlie==0:
                  feature_total=feature_total
               elif charlie==openc2_query[0]:
                  feature_total=1
               elif charlie==openc2_query[1]:
                  feature_total=feature_total+2
               elif charlie==openc2_query[2]:
                  feature_total=feature_total+4
               elif charlie==openc2_query[3]:
                  feature_total=feature_total+8
               else:
                  not_parseable()
            if feature_total==0:         
               server_200(complete_response,json.loads('{"status":200}'))
            elif feature_total==1:
               server_200(complete_response,json.loads('{"status":200,"results":{"versions":["1.0"]}}'))  
            elif feature_total==2:
               server_200(complete_response,json.loads('{"status":200,"results":{"profiles":["slpf"]}}'))
            elif feature_total==3:
               server_200(complete_response,json.loads('{"status":200,"results":{"versions":["1.0"],"profiles":["slpf"]}}'))
            elif feature_total==4:
               server_200(complete_response,json.loads('{"status":200,"results":{"pairs":{"allow":["ipv4_net"],"deny":["ipv4_net"],"query":["features"]}}}'))
            elif feature_total==5:
               server_200(complete_response,json.loads('{"status":200,"results":{"versions":["1.0"],"pairs":{"allow":["ipv4_net"],"deny":["ipv4_net"],"query":["features"]}}}'))
            elif feature_total==6:
               server_200(complete_response,json.loads('{"status":200,"results":{"profiles":["slpf"],"pairs":{"allow":["ipv4_net"],"deny":["ipv4_net"],"query":["features"]}}}'))
            elif feature_total==7:
               server_200(complete_response,json.loads('{"status":200,"results":{"versions":["1.0"],"profiles":["slpf"],"pairs":{"allow":["ipv4_net"],"deny":["ipv4_net"],"query":["features"]}}}'))
            elif feature_total==8:
               server_200(complete_response,json.loads('{"status":200,"results":{"rate_limit":10}}'))
            elif feature_total==9:
               server_200(complete_response,json.loads('{"status":200,"results":{"versions":["1.0"],"rate_limit":10}}'))
            elif feature_total==10:
               server_200(complete_response,json.loads('{"status":200,"results":{"profiles":["slpf"],"rate_limit":10}}'))
            elif feature_total==11:
               server_200(complete_response,json.loads('{"status":200,"results":{"versions":["1.0"],"profiles":["slpf"],"rate_limit":10}}'))
            elif feature_total==12:
               server_200(complete_response,json.loads('{"status":200,"results":{"pairs":{"allow":["ipv4_net"],"deny":["ipv4_net"],"query":["features"]},"rate_limit":10}}'))
            elif feature_total==13:
               server_200(complete_response,json.loads('{"status":200,"results":{"versions":["1.0"],"pairs":{"allow":["ipv4_net"],"deny":["ipv4_net"],"query":["features"]},"rate_limit":10}}'))
            elif feature_total==14:
               server_200(complete_response,json.loads('{"status":200,"results":{"profiles":["slpf"],"pairs":{"allow":["ipv4_net"],"deny":["ipv4_net"],"query":["features"]},"rate_limit":10}}'))
            elif feature_total==15:
               server_200(complete_response,json.loads('{"status":200,"results":{"versions":["1.0"],"profiles":["slpf"],"pairs":{"allow":["ipv4_net"],"deny":["ipv4_net"],"query":["features"]},"rate_limit":10}}'))
            else:
               not_parseable()
      # action is either deny, allow, update, or delete
      elif parsed_json.get("action")==openc2_actions[3]:
         not_implemented()
      elif parsed_json.get("action")==openc2_actions[4]:
         not_implemented()
      elif parsed_json.get("action")==openc2_actions[2]:
         #we are dealing with an allow action
         not_implemented()
         sanitize(net)
         execute_iptables(requested_action,net,directionality)
      elif parsed_json.get("action")==openc2_actions[1]:
         #we are dealing with an deny action
         try:
            directionality=parsed_json.get("args").get("slfp").get("direction")
         except:
            directionality="both"
         net=parsed_json.get("target").get("ipv4_net")
         requested_action=parsed_json.get("action")
         sanitize(net)
         execute_iptables(requested_action,net,directionality,'none')
      # action not found, return error
      else:
         server_error()
# tell producer we do not implement or support the request or command
def not_implemented():
   print('Status: 501 Internal Server Error')
   print('Content-Type: application/openc2-rsp+json;version=1.0')
   print('')
   print(json.dumps({'status': (501), 'status_text': 'Server error, not implemented'}, sort_keys=True, indent=4))
   print('')
   exit()

# tell producer something went wrong
def server_error():
   print('Status: 500 Internal Server Error')
   print('Content-Type: application/openc2-rsp+json;version=1.0')
   print('')
   print(json.dumps({'status': (500), 'status_text': 'Server error'}, sort_keys=True, indent=4))
   print('')
   exit()

# tell producer we could not interpret request
def not_parseable():
   print('Status: 400 Bad Request')
   print('Content-Type: application/openc2-rsp+json;version=1.0')
   print('')
   print(json.dumps(['response', {'status': (400), 'status_text': 'unable to parse request'}], sort_keys=True, indent=4))
   print('')
   exit()

# tell producer things were ok
def server_200(complete,resultant):
   print('Content-Type: application/openc2-rsp+json;version=1.0')
   print('')
   if complete==True:
      print(json.dumps(resultant, sort_keys=True, indent=4))
   else:
      resultant='{"status":200}'
      print(json.dumps(resultant, sort_keys=True, indent=4))

# function to sanitize untrusted input
def sanitize(ip_network):
   if len(ip_network)>15:
      not_parseable()
   elif len(ip_network)<7:
      not_parseable()
   else:
      sanitized=1

# function to run iptables
# presently only supports denying or allowing a source IP 
def execute_iptables(action,network,direction,expected_response):
   if action=="deny":
      action_cli="DROP"
   elif action=="allow":
      action_cli="ACCEPT"
   elif action=="delete":
      action_cli="-D"
   else:
      server_error()
   if direction=="ingress":
      chain="INPUT"
   elif direction=="egress":
      chain="OUTPUT"
   else:
      chain="BOTH"
   # iptables command to review source IP and either allow or deny it
   if chain=="BOTH":
      #need to execute two iptables commands
      #command_line_args=f"/sbin/iptables --append INPUT --source {source} --jump {action_cli}"
      result=subprocess.run(['sudo','/sbin/iptables','--append','INPUT','--source',network,'--jump',action_cli], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
      result=subprocess.run(['sudo','/sbin/iptables','--append','OUTPUT','--destination',network,'--jump',action_cli], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
      if expected_response=='complete':
         server_200(complete_response,json.loads('{"status":200,"results":{"versions":["1.0"],"profiles":["slpf"],"rate_limit":10}}'))
      else:
         server_200(complete_response,json.loads('{"status":200}'))

if __name__ == "__main__":
    main()
