---
title: Simpletalk.h
permalink: /contributions/simpletalk.h/
categories: 
  - Library Contributions
---

 `Simpletalk.h` is a modified version of Robb Sherwin's
port of Adam Cadre's Inform 6 Phototalk extension. Among other things,
it has been made into the format of a library contribution for easy
inclusion. You can download it
[here](https://www.ifarchive.org/if-archive/programming/hugo/library/contributions/simpletalk.zip)

### Special notes

Usage is explained in the comments in simpletalk.h, but you can also
take a look at the included sample "game".

### The extension itself

    !\-----------------------------------------------------------------------
    Simpletalk.h version 2.5, based on code written by Robb Sherwin, updated and
    turned into an includable library by Roody Yogurt.

    This is a simplified version of Sherwin's Phototalk port. This one drops support
    for contextualized multiple-choice menus (which it seems the original Inform 6
    version had) and is just a bare-bones list-all-available-quips system.

    changelog
       v 2.5 - Added DoQuickTalk routine to allow for commands like "talk <char> 1"
               so walkthroughs and such can work with the talk system. to use, add
               grammar like this to your game:
    verb "t","talk
       * living "1"/"2"/"3"/"4"        DoQuickTalk
       * number                        DoQuickTalk

       v 2.4 - removed unnecessary word[2] check to allow command shortening
       v 2.3 - fixed a message routine bug
       v 2.2 - added event_flag check so external things can kick the player out of
       conversation loop
       V 2.1 - added can_quit and loop_talk globals. can_quit defaults to true
       but if set false, players can't quit out of conversations. loop_talk
       defaults to false but if set true, loops conversation menus while options are
       available.

       In games with non-disappearing conversation options, authors should be wary
       of loop_talk and no-can_quit combinations.


    Usage notes:

    Of course, #include this from your main file.

    Also, add the following:

    !----------------------------------------------------------------------------
    ! Set up the quip declarations
    array qflag[100]          ! where 100 = total number of quips in the game
    array quips[5] = 20, 20, 20, 20, 20 ! 5 = number of characters, the 20s are
                                        ! how many quips are alloted to each

    Change that as needed to fit your code.

    Give characters that you can talk to in the game the property "charnumber" and
    a value starting at 1 (first character gets 1, 2nd character is 2, and so on)

    Also, add the following routines:

    routine SayQ (char, line)
    {
       local dont_remove
       select(char)
       case 1
       {
          select(line)
          case 0: {">This is the first thing you can say to char one."}
          case 1: {">This is the second thing you can say to char one."}
       }
       case 2
       {
          select(line)
          case 0: {">This is the first thing you can say to char two."}
          case 1: {">This is the second thing you can say to char two."}
       }
       if not dont_remove
          SetQuip(char,line,0,1)
       else
          SetQuip(char,line,1,1)
    }

    routine Respond (char, line)
    {
       select(char)
       case 1
       {
          select(line)
          case 0: "This is the first response from character number one."
          case 1: "This is the second response from character number one."
       }
       case 2
       {
          select(line)
          case 0: "This is the first response from character number two."
          case 1: "This is the second response from character number two."
       }
    ! Let's turn off whichever quip we selected
       SetQuip(char,line,0)
    }

       And fill them to suit your game.

       Make another routine:

    routine SetUpQuips
    {
       SetQuip(1,0,1)
       SetQuip(1,1,1)
       SetQuip(2,0,1)
       SetQuip(2,1,1)
    }

       And add SetUpQuips to the init routine in your game. This sets up all of the
       quips available at the start of the game. The first number in SetQuip is the
       character's charnumber, the second is the line that's now available through
       SayQ, and the third should just be 1 (turning the question on).

       Call SetQuip(#,#,1) to turn on other quips as the game progresses.
    -----------------------------------------------------------------------\!

    #ifclear _SIMPLETALK_H
    #set _SIMPLETALK_H

    #ifset VERSIONS
    #message "Simpletalk.h version 2.5"
    #endif

    #ifset USE_EXTENSION_CREDITING
    #ifclear _ROODYLIB_H
    #message error "Extension crediting requires \"roodylib.h\". Be sure to include it first!"
    #endif
    version_obj simpletalk_version "SimpleTalk Version 2.5"
    {
       in included_extensions
       desc_detail
          " by Robb Sherwin and Roody Yogurt";
    }
    #endif

    property charnumber

    global loop_talk
    global can_quit = true
    constant NOT_AVAILABLE 0
    constant AVAILABLE 1
    constant SPOKEN 2

    replace DoTalk
    {
       speaking = 0
       if (not xobject and word[2] ~= "about") or
          (xobject >= 1 and xobject <= 4)
       {
          if object is unfriendly
          {
             if not object.ignore_response
                Message(&Speakto, 4)    ! "X ignores you."
             !	speaking = 0
          }
          elseif object = player
          {
             Message(&Speakto, 1)    ! "Stop talking to yourself."
             return false
          }
          elseif object is not living
          {
             PhotoMessage(&DoTalk, 1) ! "you can't talk to that"
             return false
          }
          else
          {
             local a,b
             speaking = object ! this speech system doesn't really need to keep
                         ! track of who's speaking
             while true
             {
                b = Phototalk
                a = higher(a,b)
                xobject = 0
                if loop_talk and (not MoreTalk or not b)
                   break
                elseif not loop_talk
                   break
                main
                if event_flag
                {
                   select event_flag
                      case 1
                      {
                         PhotoMessage(&DoTalk,2) ! "Do you want to keep talking?"
                         GetInput
                         if not YesOrNo
                         {
                            break
                         }
                      }
                         case 2: break
                }
                ""
             }
    #ifclear NO_SCRIPTS
             if a        ! only successful conversations pause scripts
                SkipScript(object)
    #endif
          }
          return true
       }

       PhotoMessage(&DoTalk,3) ! "Just talking to <the object> will suffice."
       return false
    }

    routine Phototalk
    {
       local x, y, z, ok = 0
       local selected

    ! Count up all the lines by previous characters.
       if object.charnumber
       {
          for (x=0; x < object.charnumber; x++)
          {
             y += quips[x]
          }
       }

    ! Check and make sure you have something to say.
       for (x=y; x<(y+quips[object.charnumber]); x++)
       {
          if (QuipOn(object,x-y))
          {
             ok++
    #ifclear AUTOMATIC_SAY
             if not xobject
                break
    #endif
          }
       }

    ! Write contents to the screen
       if ok
       {
    #ifset AUTOMATIC_SAY
          if ok = 1 and not xobject ! and not BeenSpoken(object,t)
             xobject = 1
    #endif
          if not xobject or xobject < 0 or xobject > ok
          {
             PhotoMessage(&PhotoTalk, 1) ! "Please select one:"
             ""
          }

          ! List the player's choices
          for (x=y;  x < (y+(quips[object.charnumber])); x++)
          {
             if (QuipOn(object,x-y))
             {
                z++
                if not xobject
                   PhotoMessage(&PhotoTalk, 2, z) ! "(#)"
                if not xobject or xobject = z
                   SayQ(object.charnumber,x-y)
             }
          }

          if not xobject
          {
             ""
             while true
             {
                selected = GetDial(z)
                if can_quit or (not can_quit and selected)
                   break
             }
          }
          else
             selected = xobject

          ""
          if (selected ~= 0)
          {
             ok=0
             for (x=y;  x < (y+quips[object.charnumber]); x++)
             {
                if (QuipOn(object,x-y))
                {
                   ok++
                   if (ok = selected)
                   {
                      Respond(object.charnumber,x-y)
                   }
                }
             }
          }
          else
          {
          PhotoMessage(&Phototalk,3) ! "Eeeagh! Stage fright! Abort!"
          }

          return selected
       }

       PhotoMessage(&PhotoTalk,4) ! "You really have nothing to say right now."
    }

    routine DoQuickTalk
    {
       local r
       local i = 1
       while word[(i+1)] ~= ""
       {
          i++
       }
          if i > 2
          {
             select word[i]
                case "1": r = 1
                case "2" : r = 2
                case "3" : r = 3
                case "4" : r = 4
          }

       if not r
       {
          if speaking
          {
             r = object
             object = speaking
          }
          else
          {
             "It is unclear whom you are speaking to."
             return
          }
       }
       else
          speaking = object
       return Perform(&DoTalk, speaking, r)
    }

    routine MoreTalk
    {
       local x, y

       if object.charnumber
       {
          for (x=0; x < object.charnumber; x++)
          {
             y += quips[x]
          }
       }

       for (x=y; x<(y+quips[object.charnumber]); x++)
       {
          if (QuipOn(object,x-y)) ! and not BeenSpoken(object, x-y)
          {
             return true
          }
       }
    }

    routine GetDial(max)
    {
       word[1] = ""
       local temp
       PhotoMessage(&GetDial,1) ! "Select a choice or 0 to keep quiet. >> "
       input

       while true
       {
          if word[1] = "0" or words = 0
             break
          if word[1]
             temp = StringToNumber(word[1])
          else
             temp = StringToNumber(parse$)
          if temp ~= 0 and temp <= max
             break
          PhotoMessage(&GetDial,1) ! "Select a choice or 0 to keep quiet. >> "
          input
          temp = 0
       }

       return temp
    }

    routine SetQuip (char, line, onoff, markused)
    {
       local x n

       for (x=0; x<char; x++)
       {
          n += quips[x]
       }

       n += line

       if markused
       {
          qflag[n] = onoff | SPOKEN
       }
       else
          qflag[n] = onoff
    }

    routine QuipOn (char, line)
    {
       local x n

       for (x=0; x<char.charnumber; x++)
       {
          n += quips[x]
       }
       n += line
       if (qflag[n] & AVAILABLE)
          return true
    }

    routine BeenSpoken(char, line)
    {
       local x n

       for (x=0; x<char.charnumber; x++)
       {
          n += quips[x]
       }
       n += line
       if (qflag[n] & SPOKEN)
          return true
    }

    replace DoAsk
    {
       return PrintConverseUsage
    }

    replace DoAskQuestion
    {
       return PrintConverseUsage
    }

    replace DoTell
    {
       return PrintConverseUsage
    }

    routine PrintConverseUsage
    {
       "Try \B>TALK TO CHARACTER\b instead."
       return false
    }

    routine PhotoMessage(r, num, a, b)
    {
       local ret
       ret = NewPhotoMessages(r, num, a, b)
       if ret : return ret

       select r
          case &PhotoTalk
          {
             select num
                case 1: "Please select one:"
                case 2
                {
                   print "("; number a; ")";
                }
                case 3: "Eeeagh! Stage fright! Abort!"
                case 4: "You really have nothing to say right now."

          }
          case &GetDial
          {
             select num
                case 1
                {
                   if can_quit
                      "Select a choice or 0 to keep quiet. >> ";
                   else
                      "Select a choice>> ";
                }
          }
          case &DoTalk
          {
             select num
                case 1: "You can't talk to that!"
                case 2
                {
                   print "\nDo you want ";
                   if player_person ~= 2:  print The(player, true); " ";
                   print "to keep talking (YES or NO)? ";
                }
                case 3: 	print "Just talking to "; art(object); " will suffice."
          }
    }

    !\ The NewPhotoMessages routine may be REPLACED and should return
    true if a replacement message exists for routine <r> \!

    routine NewPhotoMessages(r, num, a, b)
    {
       select r
       case else : return false
       return true
    }

    #endif


