```matlab
%% PROTOCOL: Echange de clé
%%
%%
%%
%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%
%% définition du rôle Alice, initiant le protocole
%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

role alice (A, B, TS: agent,             
            PKa, PKb: public_key,
            KAts: symmetric_key,    
            SND, RCV: channel(dy)) 
played_by A def=

  local State: nat, 
        Na, Nb: text,
        PKaBIS, PKbBIS: public_key,
        CleSession, KBts : symmetric_key

  const ok: text

  init State:=0

  transition  
   
    01.  State=0 /\ RCV(start) =|>
            State':=1 /\
            Na':=new() /\
            secret(Na', na, {A,B}) /\ 
            SND(A.{B.Na'}_KAts)


    06. State=1 /\ RCV({CleSession'.{Nb'}_KBts}_KAts) =|>
            State':=2 /\
            request(A, B, a_b_CleSession, CleSession') /\
            SND({{Nb'}_KBts}_CleSession')

    08. State=2 /\ RCV({{Na}_KAts}_CleSession) =|>
            State':=3



end role


%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%
%% définition du rôle Bob
%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

role bob (A, B, TS: agent,             
            PKa, PKb: public_key,
            KBts: symmetric_key,    
            SND, RCV: channel(dy)) 
played_by B def=

  local State: nat, 
        Na, Nb: text,
        PKaBIS, PKbBIS: public_key,
        CleSession, KAts: symmetric_key

  init State:=0

  transition  
   
    03. State=0 /\ RCV({A}_KBts) =|> 
            State':=1 /\
            Nb':=new() /\
            secret(Nb', nb, {A,B}) /\ 
            SND({Nb'}_KBts)

    05. State=1 /\ RCV({CleSession'.{Na'}_KAts}_KBts) =|>
            State':=2 /\
            request(A, B, a_b_CleSession, CleSession')

    07. State=2 /\ RCV({{Nb}_KBts}_CleSession) =|>
            State':=3 /\
            SND({{Na}_KAts}_CleSession)



end role





%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%
%% définition du rôle Serveur de Confiance
%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

role ts (A, B, TS: agent,             
            PKa, PKb: public_key,
            KAts, KBts: symmetric_key,
            SND, RCV: channel(dy))  
played_by TS def=

  local State: nat, 
        Na, Nb: text,
        PKaBIS, PKbBIS: public_key,
        CleSession: symmetric_key

  const ok: text

  init State:=0

  transition  
   
    02. State=0 /\ RCV(A.{B.Na'}_KAts) =|>
            State':=1 /\
            SND({A}_KBts)

    04. State=1 /\ RCV({Nb'}_KBts) =|> 
            State':=2 /\
            CleSession':=new() /\
            secret(CleSession', cleSession, {A,B}) /\ 
            witness(A, B, a_b_CleSession, CleSession') /\
            SND({CleSession'.{Nb'}_KBts}_KAts) /\
            SND({CleSession'.{Na}_KAts}_KBts)


end role




%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%
%% définition du rôle Session
%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

role session(A, B, TS: agent, PKa, PKb: public_key, KAts, KBts: symmetric_key) def=

  local SA, RA, SB, RB, SS, RS: channel(dy)
 
  composition 

        alice(A,B,TS,PKa,PKb,KAts,SA,RA) /\ 
        bob(A,B,TS,PKa,PKb,KBts,SB,RB) /\
        ts(A,B,TS,PKa,PKb,KAts,KBts,SS,RS)
end role


%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%
%% définition du Scenario
%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

role environment() def=


    const a, b, ts: agent,
          pka, pkb, pki: public_key,
          cleSession: symmetric_key, 
          kats, kbts : symmetric_key,
          na, nb, a_b_CleSession : protocol_id,
          h : hash_func


    intruder_knowledge = {a, b, ts, pka, pkb, pki, inv(pki), h}


    composition

        session(a,b,ts,pka,pkb,kats,kbts)

end role

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%
%% définition des Propriétés à vérifier
%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

goal
    secrecy_of na
    secrecy_of nb
    secrecy_of cleSession
    authentication_on a_b_CleSession
end goal


%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%
%% lancement du rôle principal
%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

environment()
```